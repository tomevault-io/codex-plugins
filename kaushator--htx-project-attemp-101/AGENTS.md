# Copilot Instructions for HTX Project

## Overview
- This is a multi-component trading analytics platform for HTX (Huobi) and 3Commas, with a FastAPI async backend, React frontend, and SQLAlchemy 2.0 async DB layer (SQLite/PostgreSQL).
- Major backend domains: API endpoints, async DB models, file parsing, PnL analytics, cashflow, and external integrations (HTX, 3Commas).
- Data flows: CSV/Excel files are parsed, stored in DB, analyzed, and exposed via REST API. 3Commas/HTX APIs are used for sync and automation.

## Key Directories & Files
- `backend/app/` — FastAPI app, all main logic
  - `main.py` — FastAPI entrypoint, CORS, routers, error handling
  - `core/` — config, logging, settings
  - `api/v1/endpoints/` — REST endpoints (health, files, trades, cashflow, pnl)
  - `models/`, `schemas/` — SQLAlchemy models, Pydantic schemas
  - `services/` — business logic, integrations (e.g. `threecommas.py`)
  - `db/` — async DB session, migrations
- `frontend/` — React app (optional)
- `htx_project/src/` — legacy/CLI analytics, file parsing, integrations
- `data/` — raw/processed files
- `scripts/` — setup, push, log (see `.vscode/tasks.json`)

## Developer Workflows
- **Setup:**
  - Windows: `scripts/setup_env.bat` (creates `.venv`, installs deps, runs backend)
  - Manual: `python -m venv .venv && .venv\Scripts\activate && pip install -r requirements.txt`
- **Run backend:**
  - From `backend/`: `python -m uvicorn app.main:app --host 127.0.0.1 --port 8004`
- **Testing:**
  - From `backend/`: `pytest`
- **Lint/Format:**
  - `black .`, `flake8 .`, `mypy .`
- **Migrations:**
  - Alembic config in `backend/alembic.ini`, migrations in `backend/alembic/`
- **Frontend:**
  - Usual React workflow (`npm install`, `npm run dev`)

## Patterns & Conventions
- All backend imports are absolute from `app.` (e.g. `from app.core.config import settings`)
- Async everywhere: DB, endpoints, services
- Pydantic for config and validation
- Logging via `core/logging.py`, logs to `logs/htx_project.log`
- API versioning: all endpoints under `/api/v1/`
- Integrations: see `services/threecommas.py`, `src/integrations/three_commas.py`
- Tests in `backend/tests/`, run with `pytest`

## Integration & External
- HTX API: keys in `config.yaml` or `.env`
- 3Commas: keys in `config.yaml` or `.env`, logic in `services/threecommas.py`
- File uploads: via `/api/v1/files/upload`, requires `python-multipart`

## Examples
- Add new endpoint: create router in `api/v1/endpoints/`, include in `main.py`
- Add new service: implement in `services/`, inject via endpoint
- Add DB model: define in `models/`, migrate with Alembic

---
For more, see `README.md` and `docs/architecture.md`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kaushator)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Kaushator)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
