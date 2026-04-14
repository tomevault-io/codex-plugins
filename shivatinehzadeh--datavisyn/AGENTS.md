## Purpose

Short, actionable instructions for AI agents working in this repository. Focus on the live FastAPI service, DB access patterns, and how the project is run locally and in Docker.

## Quick architecture (what to know fast)
- This project exposes a FastAPI app in `gene_api/core/base.py`. The Docker production command runs `uvicorn core.base:app`.
- Database layer lives in `gene_api/core/db_setup.py`. It uses SQLAlchemy (declarative Base), a session factory `session_local`, and a dependency generator `get_db()` that yields DB sessions.
- Routers are registered on the FastAPI app using the pattern `app.include_router(<module>.router, prefix="/api")`. Example: `app.include_router(auth.router, prefix="/api")` in `core/base.py`.

## Key files to inspect
- `gene_api/core/base.py` — application entrypoint & router inclusion.
- `gene_api/core/db_setup.py` — DB URL builder, engine, `get_db()` dependency.
- `DockerFile` — image build, test and production targets (production runs uvicorn on `core.base:app`).
- `docker-compose.yml` — services used for local development: `db`, `fastapi`, and `test`. Useful for integration/dev workflows.
- `requirements.txt` — primary Python deps (FastAPI, uvicorn, SQLAlchemy, alembic, pytest, httpx, python-dotenv).

## Environment & runtime
- DB connection relies on these environment variables (read in `db_setup.py`):
  - POSTGRES_USER
  - POSTGRES_PASSWORD
  - POSTGRES_HOST
  - POSTGRES_DB
  - POSTGRES_PORT
- The project uses `python-dotenv` so a `.env` file in the repo root is expected for local development.

## How to run & test (concrete commands)
- Run locally (development):
  - Install deps: `pip install -r requirements.txt` (from repo root)
  - Start with auto-reload: `uvicorn core.base:app --reload --host 127.0.0.1 --port 8000`
- Run in Docker Compose (recommended for DB):
  - `docker-compose up --build` (will start `db` and `fastapi`)
- Run tests:
  - `pytest -vv` (Docker has a `test` service that runs this as well)

## Code patterns & conventions (be specific)
- Routers: expect modules that expose a `router` object (FastAPI APIRouter). Include them from `core/base.py` using `app.include_router(...)` with a `/api` prefix.
- DB sessions: functions that need DB access should accept a `db` dependency via `Depends(get_db)` from `gene_api/core/db_setup.py` and must not commit/close the session manually (the dependency handles closing).
- Models: use SQLAlchemy declarative `Base` from `db_setup.py` and `session_local` for session creation in tests or scripts.
- Tests: use `pytest` and `httpx` for HTTP client tests. Keep tests isolated (use a test DB or mocking).

## Integration points and external dependencies
- PostgreSQL is expected (see docker-compose). Connection string built in `db_setup.py`.
- Alembic is listed as a dependency — look for migration tooling before adding schema changes. If no alembic directory exists, add migrations under `alembic/` and wire `alembic.ini`.

## When editing code, be careful to:
- Preserve FastAPI app import path `core.base:app` used by Dockerfile.
- Keep router names and `router` exports consistent so `app.include_router(...)` remains valid.
- Use `get_db()` for DB access to avoid leaking sessions.

## Examples to copy/paste
- Add a router to `core/base.py`:

  from datavisyn_project.app import auth  # module that exposes `router`
  app.include_router(auth.router, prefix="/api")

- Use DB dependency in an endpoint:

  from fastapi import Depends
  from datavisyn_project.core.db_setup import get_db

  def endpoint(db: Session = Depends(get_db)):
      # use `db` (SQLAlchemy session)
      ...

## What not to change without asking
- Do not rename `core.base:app` or change Dockerfile service names without updating `docker-compose.yml`.

## Verification steps after changes
- Run `pytest -vv` locally.
- Start the stack with `docker-compose up --build` and check the FastAPI root on `http://localhost:8000`.

---
If anything here is unclear or you want more detail for a specific area (routing conventions, DB migration workflow, or example tests), tell me which section to expand and I will iterate.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shivatinehzadeh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
