# D&D Tools - AI Coding Agent Instructions

## Architecture Overview

This is a Python FastAPI application for D&D campaign management with AI-powered content generation.

**Core components:**
- `src/web/app.py` - FastAPI routes with HTMX-powered UI (campaigns, artifacts, background jobs)
- `src/web/db.py` - SQLite database layer with dataclasses (`Campaign`, `Artifact`, `Job`)
- `src/ai_tools/generator.py` - `DnDGenerator` class wrapping OpenAI SDK (LiteLLM-compatible)
- `src/workflows/campaign_pack.py` - AI-driven campaign generation with flowchart export
- `src/image_generation/comfyui.py` - ComfyUI integration for image generation

**Data flow:** Web UI → FastAPI routes → DB (SQLite WAL mode) + AI generators → Artifacts stored in `data/app/campaigns/{id}/`

## Developer Commands

Use [Task](https://taskfile.dev/) for all operations:
```bash
task deps          # Install/sync Python deps via uv
task test          # Run pytest
task web:dev       # Local dev server with hot reload
task stack         # Full Docker stack (web + LiteLLM + Ollama)
task litellm       # LiteLLM proxy only (for hybrid local dev)
```

## Project Conventions

### Package Management
- Uses `uv` (not pip/poetry). Run scripts with `uv run python ...`
- Dependencies in `pyproject.toml`, no requirements.txt

### AI/LLM Integration
- All LLM calls go through `DnDGenerator` which auto-detects LiteLLM proxy via `LITELLM_BASE_URL`
- Default model: `gpt-5.2` (configurable via `DND_DEFAULT_MODEL` env var)
- LLM responses requiring JSON use `_extract_json()` helper in `campaign_pack.py`

### Web Layer Patterns
- HTMX partials in `templates/partials/` for dynamic updates
- Background tasks via FastAPI's `BackgroundTasks` with job status polling
- Artifact types: `text.*` (stored in DB), `file.*` (stored on disk at `campaign_artifacts_dir()`)
- Use `db.create_artifact()` / `db.create_job()` for new content types

### Data Storage
- SQLite at `data/app/app.db` with foreign keys enabled
- Per-campaign directories: `data/app/campaigns/{campaign_id}/artifacts/` and `/uploads/`
- Paths resolved via `src/web/paths.py` helpers (e.g., `campaign_artifacts_dir(id)`)

### Testing
- Tests in `tests/` using pytest
- Run with `task test` or `uv run pytest -q`
- Tests verify imports and CLI tools, not full integration

## Key Files for New Features

| Feature Area | Key Files |
|-------------|-----------|
| New AI generator | `src/ai_tools/generator.py` (add method to `DnDGenerator`) |
| New web endpoint | `src/web/app.py`, `src/web/db.py`, `templates/partials/` |
| New artifact type | `src/web/db.py` (schema), `app.py` (routes) |
| Image generation | `src/image_generation/comfyui.py` |
| Campaign workflows | `src/workflows/campaign_pack.py` |

## Environment Variables

Required in `.env`:
- `OPENAI_API_KEY` or `ANTHROPIC_API_KEY` - for AI features
- `LITELLM_BASE_URL` - set automatically in Docker, optional for local dev
- `DND_DEFAULT_MODEL` - override default LLM model
- `COMFYUI_BASE_URL` - for image generation features

## Local Network Deployment (k3s)

The project supports deployment on a local k3s cluster with GPU-accelerated AI services.

### Service URLs (k3s network)
Configure these environment variables to point to your k3s services:
- `LITELLM_BASE_URL=http://<k3s-node>:4000` - LiteLLM proxy routing to Ollama/OpenAI/Anthropic
- `OLLAMA_API_BASE=http://<k3s-node>:11434` - Direct Ollama access (used by LiteLLM)
- `COMFYUI_BASE_URL=http://<k3s-node>:8188` - ComfyUI for Stable Diffusion image generation

### Ollama on k3s
- Exposes OpenAI-compatible API on port 11434
- LiteLLM routes `ollama/*` models (e.g., `llama3.3`) via `litellm.yaml` config
- Pull models: `kubectl exec -it <ollama-pod> -- ollama pull llama3.3`
- Set `DND_DEFAULT_MODEL=llama3.3` for fully local LLM inference

### ComfyUI / Stable Diffusion on k3s
- Exposes REST API on port 8188 for txt2img workflows
- `src/image_generation/comfyui.py` provides `build_txt2img_workflow()`, `queue_prompt()`, `wait_for_result_image()`
- Requires GPU node with NVIDIA runtime; mount models at `/app/models`
- Checkpoint files (SD1.5, SDXL) go in the `checkpoints/` subfolder
- For ~8GB VRAM, prefer SD1.5 or SDXL-turbo models at modest resolutions

### k3s Deployment Notes
- Use `NodePort` or `LoadBalancer` services to expose Ollama/ComfyUI/LiteLLM on LAN
- GPU workloads require `nvidia.com/gpu` resource limits and k3s device plugin
- Persistent volumes recommended for Ollama models (`/root/.ollama`) and ComfyUI outputs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/whatnick) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
