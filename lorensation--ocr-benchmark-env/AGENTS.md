# Copilot Instructions: doc-benchmark-deepseek-ocr

Portfolio-grade OCR benchmarking platform comparing DeepSeek-OCR and Tesseract in a multi-service Docker architecture.

## Architecture Overview

**Multi-service system** with clear separation:
- `services/api/` — FastAPI orchestrator handling uploads, coordinating OCR services, storing results
- `services/deepseek_service/` — DeepSeek-OCR inference server (HuggingFace Transformers)
- `services/tesseract_service/` — Tesseract OCR baseline service
- `services/benchmark_worker/` — Metrics computation (currently unused, results computed inline in API)
- `hf_space/` — Standalone Streamlit demo for Hugging Face Spaces deployment

**Communication pattern**: HTTP-based inter-service calls via Docker network URLs (`http://deepseek:9000/ocr`, `http://tesseract:9001/ocr`). API uses `requests` with retry logic (exponential backoff, 3 max retries).

**Data flow**: Upload → API validates file → Saves to `data/uploads/` → Calls both OCR services → Aggregates results → Stores JSON in `data/results/{run_id}.json`.

## Critical Conventions

### Configuration Pattern
All services use **Pydantic Settings** for config validation. Example from `api/main.py`:
```python
class Settings(BaseSettings):
    upload_dir: str = Field(default="/app/data/uploads")
    deepseek_url: str = Field(default="http://deepseek:9000/ocr")
    max_file_size_mb: int = Field(default=10)
    
    class Config:
        env_file = ".env"
        case_sensitive = False
```
Environment variables are **always lowercase** in `.env` but accessed as snake_case attributes.

### Structured Logging
JSON-structured logs for all services. Each service prefixes with service name:
```python
logging.basicConfig(
    format='{"time": "%(asctime)s", "level": "%(levelname)s", "message": "%(message)s", "service": "api"}'
)
```
Use correlation IDs (run_id) to trace requests across services.

### Error Handling Pattern
- **Validation errors**: HTTP 400 with specific error message
- **Timeout errors**: HTTP 504 Gateway Timeout after 30s (configurable)
- **Service unavailable**: HTTP 503 when OCR services are down
- **File too large**: HTTP 413 (10MB default limit)

Every API endpoint wraps external calls in try/except with proper HTTP status codes. See `call_ocr_service()` in `api/main.py` for the reference pattern.

### File Security & Validation
Three-stage validation in `validate_file()`:
1. **Extension check**: Against whitelist (png, jpg, jpeg, gif, bmp, tiff, webp)
2. **MIME type validation**: Using `mimetypes.guess_type()`
3. **Filename sanitization**: `re.sub(r'[^\w\s\-\.]', '', filename)`

Files saved as `{run_id}_{sanitized_filename}` to prevent collisions.

## Development Workflows

### Local Testing
```bash
# Full stack (slow, loads DeepSeek model ~5min)
docker-compose up --build

# Fast testing (Tesseract only)
docker-compose -f docker-compose.test.yml up --build

# Unit tests (no services needed)
pytest -m unit

# Integration tests (requires running services)
pytest -m integration
```

### Test Organization
- Tests use **pytest markers**: `@pytest.mark.unit`, `@pytest.mark.integration`, `@pytest.mark.api`
- Fixtures in `tests/conftest.py` provide: `temp_upload_dir`, `temp_results_dir`, `sample_image_bytes`
- Mock pattern for API tests: `monkeypatch.setenv()` to override settings before importing `main.py`
- Coverage target: **80%** (configured in `pytest.ini`)

### Docker Health Checks
All services expose `/health` and `/ready` endpoints:
- `/health` — Basic liveness check
- `/ready` — Service readiness (e.g., DeepSeek model loaded)

Compose healthchecks use `curl -f http://localhost:{port}/health` with retries and start_period.

## Service-Specific Guidance

### API Service (`services/api/`)
- Uses `requests.Session()` with retry strategy (see `create_retry_session()`)
- All endpoints return Pydantic models (`BenchmarkResult`, `RunListItem`, `ErrorResponse`)
- CORS enabled for all origins (frontend development)
- Results stored as individual JSON files, not database (lightweight, git-friendly)

### DeepSeek Service (`services/deepseek_service/`)
- Model loading in **lifespan context manager** (startup/shutdown events)
- CPU-only: `AutoModel.from_pretrained()` without GPU kwargs
- Inference pattern:
  ```python
  result = model.infer(
      tokenizer,
      prompt="<image>\n<|grounding|>Convert the document to markdown.",
      image=pil_image,
      base_size=1024,
      crop_mode=True
  )
  ```
- Long startup time (~120s), reflected in healthcheck `start_period`

### Tesseract Service (`services/tesseract_service/`)
- Wraps `pytesseract.image_to_string()`
- Fast startup, simple stateless processing
- Error handling for corrupted images using PIL exception catching

### HF Space (`hf_space/app.py`)
- **Standalone deployment**: No external API dependency, loads model locally
- Uses `@st.cache_resource` to cache model loading across sessions
- Streamlit CPU execution, no GPU

## CPU-Only Constraint

**Critical**: All code must run on CPU. Never add:
- `.to("cuda")` calls
- GPU-specific dependencies (flash-attn, bitsandbytes)
- CUDA-based image libraries

Use `model.eval()` for inference mode to reduce memory.

## Current Limitations

1. **No async processing**: Benchmarks run synchronously; `benchmark_worker/` service exists but isn't used
2. **Basic metrics only**: `len(text)` comparison; no Levenshtein, CER, WER yet
3. **No batch processing**: One image per request
4. **No ground truth**: No dataset with labeled references for accuracy metrics

When suggesting improvements, reference `ROADMAP.md` Phase 2-3 goals.

## Adding New OCR Models

Follow this pattern (see existing services):
1. Create `services/{model_name}_service/` with `server.py`, `Dockerfile`, `requirements.txt`, `INSTRUCTIONS.md`
2. Add FastAPI endpoint `POST /ocr` accepting `UploadFile`, returning `{"text": str, "confidence": float}`
3. Add healthcheck endpoints `/health` and `/ready`
4. Update `docker-compose.yml` with service definition and healthcheck
5. Update `api/main.py` to call new service URL
6. Add corresponding tests in `tests/integration/`

## Key Files to Reference

- `services/api/main.py` (lines 50-100) — Retry logic and session setup
- `services/api/main.py` (lines 120-180) — Validation and error handling patterns
- `services/deepseek_service/server.py` (lines 1-60) — Model loading with error recovery
- `tests/conftest.py` — Standard test fixtures
- `pytest.ini` — Test markers and coverage config
- `ROADMAP.md` — Feature priorities and completed work

## Questions to Ask

Before implementing features, clarify:
1. Should this work with the full stack or `docker-compose.test.yml`?
2. Is this for local deployment, HF Space, or both?
3. Do we need to maintain backward compatibility with existing results JSON?
4. Should this be synchronous or prepare for future async workers?

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lorensation) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
