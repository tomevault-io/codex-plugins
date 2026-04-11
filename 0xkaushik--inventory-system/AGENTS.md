# Copilot instructions for this repository

Quick, actionable notes for an AI coding agent to be productive in this repo.

## Big picture
- Backend: a small FastAPI app using SQLAlchemy + SQLite. Entry is `backend/main.py`.
- Frontend: a Vite + React app under `frontend/frontend` (dev server on 5173).
- Data model centers on `Product -> Dimension -> SliderType/Color -> Stock` (see `backend/models.py`).

## How to run (reproducible commands)
- Start backend (from repo root):

  - `cd backend && uvicorn main:app --reload --port 8000`
  - or from repo root: `uvicorn backend.main:app --reload --port 8000`

- Seed the DB (creates / populates `backend/inventory.db`):

  - `python backend/seed.py`

- Start frontend (from `frontend/frontend`):

  - `cd frontend/frontend && npm install && npm run dev` (Vite runs on 5173)

Notes: frontend fetches the API at `http://localhost:8000` (`frontend/frontend/src/api.js`) and backend CORS allows `http://localhost:5173` (`backend/main.py`).

## Key files to inspect when changing behavior
- API surface and routing: `backend/main.py` (FastAPI endpoints).
- Business logic / DB operations: `backend/crud.py` (use this for transactional logic).
- DB models / schema: `backend/models.py` and `backend/schemas.py` (Pydantic v2 uses `from_attributes`).
- Pricing / size rules: `backend/utils.py` (pure functions used by `crud.py`).
- DB setup: `backend/database.py` and seed data in `backend/seed.py`.
- Frontend API calls: `frontend/frontend/src/api.js` and components under `frontend/frontend/src`.

## Project-specific patterns & conventions
- Pydantic models are written for v2: they set `Config.from_attributes = True` (look in `backend/schemas.py`).
- CRUD functions return ORM objects (not dicts). FastAPI uses `response_model` to convert via Pydantic.
- Size rules: `SizeRule` may set `fixed_size` OR `min_size`/`max_size`. Use `utils.generate_allowed_sizes()` to compute allowed sizes.
- Pricing: `utils.compute_price(base_price, size, base_size)` applies simple per-inch deltas; keep pricing pure and testable.
- Error handling: `crud` raises `ValueError` for domain issues (invalid size / insufficient stock); endpoints translate these to HTTP 400.

## Integration and gotchas noticed in the code
- SQLite DB path is relative: `SQLALCHEMY_DATABASE_URL = "sqlite:///./inventory.db"` in `backend/database.py`. Run commands from `backend` or adjust path.
- Duplicated code in `backend/main.py`: some endpoints and `get_db()` are defined twice. Be conservative: prefer minimal, well-tested changes and call out cleanup in PRs.
- No `requirements.txt` found. When running locally, install at least: `fastapi`, `uvicorn`, `sqlalchemy`, `pydantic` (and for frontend: `npm install`).

## Small examples for quick edits
- Add a new GET that lists product names: implement DB access in `backend/crud.py` and expose from `backend/main.py` with `response_model=list[schemas.ProductBase]`.
- To change pricing logic, edit `backend/utils.py::compute_price` and add unit tests (no tests exist currently; describe changes in PR).

## When opening PRs
- Keep backend changes minimal and isolated: modify `crud.py` for DB logic, `utils.py` for pure functions, and `main.py` only for routing.
- Mention DB migrations are manual: this repo uses SQLAlchemy create_all; if changing models, include a clear migration/seed plan.

## If something is unclear
- Ask for where to run commands (container vs host). If DB path varies, confirm intended working directory before changing `database.py`.

---
If you'd like, I can refine any section, add exact `pip` dependency pins, or create a simple `requirements.txt` and run scripts. What should I clarify or expand? 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/0xKaushik)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/0xKaushik)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
