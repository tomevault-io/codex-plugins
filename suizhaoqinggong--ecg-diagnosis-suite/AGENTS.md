# Copilot Instructions for ECG-Diagnosis-Suite

## Build, test, and lint commands

### Frontend (`frontend/`)

- Install deps: `pnpm install` (or `npm install`)
- Dev server: `pnpm dev`
- Build: `pnpm build`
- Preview build: `pnpm preview`
- Lint: `pnpm lint`

Notes:
- Vite dev server proxies `/api` to `http://localhost:8000` (`frontend/vite.config.ts`).
- In Docker/Nginx deployment, `/api` is proxied to backend service `http://backend:8000` (`frontend/nginx.conf`).

### Backend (`backend/`)

- Create venv and install:
  - `python -m venv venv`
  - `source venv/bin/activate`
  - `pip install -r requirements.txt`
- Run API locally:
  - `uvicorn app.main:app --reload --host 0.0.0.0 --port 8000`
- Run tests (from repo root):
  - `pytest`

Single-test examples:
- File: `pytest tests/test_api.py`
- Single test: `pytest tests/test_api.py::test_health_check`
- Backend ML integration script via pytest collection:
  - `pytest backend/test_resnet1d.py::test_model_creation`

Lint/type tools are installed in backend requirements and used directly:
- Format: `black app/ && isort app/`
- Type check: `mypy app/`

### Full stack with Docker

- Start: `docker-compose up -d`
- Logs: `docker-compose logs -f`
- Stop: `docker-compose down`

## High-level architecture

- The repository is a two-service app plus optional infra:
  - Frontend: React + Vite + TypeScript (`frontend/`)
  - Backend: FastAPI (`backend/`)
  - Optional infra in `docker-compose.yml`: PostgreSQL + Redis

- Request flow (main diagnosis path):
  1. User uploads ECG image in `frontend/src/components/ImageUpload.tsx`.
  2. Frontend posts multipart form data to `POST /api/diagnose`.
  3. FastAPI route in `backend/app/api/diagnosis.py` validates/saves upload under `./data/uploads`.
  4. Route currently returns a result based on in-file `SYMPTOM_DATABASE` (placeholder inference path).
  5. Frontend renders result via `frontend/src/components/DiagnosisResult.tsx`.

- Secondary ML path (conduction-disorder specialization):
  - API: `POST /api/detect/conduction-disorder` in `backend/app/api/conduction_disorder.py`.
  - It lazily initializes a global detector singleton with `create_cd_detector()`.
  - `ConductionDisorderDetector` (`backend/ml/conduction_disorder_detector.py`) converts ECG image to 12-lead 1D signal using `ECGImageToSignal` and runs `ResNet1DBaseline` (`backend/ml/resnet1d_model.py`).

- Backend composition:
  - App entry: `backend/app/main.py`
  - Settings: `backend/app/core/config.py` (Pydantic settings, `.env` loading)
  - API routers: `backend/app/api/diagnosis.py`, `backend/app/api/conduction_disorder.py`
  - PDF report generation utility: `backend/app/services/report_generator.py`
  - ORM models are defined in `backend/app/models/db_models.py` but not yet wired into API routes.

- Frontend composition:
  - App shell: `frontend/src/App.tsx`
  - Single page currently used: `frontend/src/pages/HomePage.tsx`
  - Upload/result components implement most UI behavior.

## Key conventions in this codebase

- Bilingual product language (Chinese-first with English support):
  - User-facing response fields and disclaimers are commonly Chinese strings.
  - Keep medical disclaimer text in API responses and UI when touching diagnosis-related features.

- API prefixing convention:
  - All functional endpoints are mounted under `/api` in `app.main`.
  - Frontend assumes relative `/api/...` endpoints and relies on proxying rather than hardcoding host in components.

- File-system data convention:
  - Uploads and generated artifacts are stored under repo-level `data/` (`data/uploads`, `data/reports`).
  - Multiple modules hardcode these paths (`diagnosis.py`, `conduction_disorder.py`, report generator, cleanup script); preserve consistency if refactoring.

- ML integration pattern:
  - 1D model expects `[batch, 12, 1000]` signal tensors.
  - ECG image -> signal conversion is a dedicated adapter (`ECGImageToSignal`) used by model services.
  - Specialized detector logic (risk levels/descriptions) lives in service layer, not API router.

- Current implementation maturity convention:
  - Core diagnosis route still uses placeholder `SYMPTOM_DATABASE`.
  - Additional DB and report modules exist but are partially integrated; check wiring before assuming persistence/report endpoints are complete.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/suizhaoqinggong)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/suizhaoqinggong)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
