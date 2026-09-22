# AI Coding Assistant Instructions for Dota Game Finder API

## Project Overview
FastAPI backend that fetches, caches, and scores Dota matches. Exposes endpoints for reading recent/cached matches, recalculating scores, and rating matches. Uses SQLAlchemy + Postgres/SQLite.

## Architecture
- API: FastAPI app in `app.py`
- Data access: `database.py` with `SessionLocal`, models (`MatchRating`, `CachedMatch`, `LatestMatch`), and `init_db()`
- Scoring & utils: `dota/` package (`calculate_scores.py`, `get_and_score_func.py`, `utils.py`, `api.py`)
- Config/constants: `constants.py` (active), `constants_old.py` (legacy)
- Tests: under `tests/`

## API Conventions
- Base path: `"/api"` for JSON endpoints; `/` serves a simple HTML index
- CORS: allow localhost 5173 and Vercel preview deployments (see CORS config in `app.py`)
- Health: `GET /api/health` verifies DB connectivity and masks DB URL
- Rating: `POST /api/rate_match` accepts `{ match_id: int, score: int }`

## Development Workflow
- Environment:
  - DB via `DATABASE_URL`; fallback to local SQLite if needed
  - Ensure `requests` timeouts and external API calls are guarded
- Local server: `python -m uvicorn app:app --reload --host 0.0.0.0 --port 8002`
- Run tests: `python -m pytest -q`

## Coding Conventions
- Type hints for FastAPI endpoints and helper functions
- Use dependency-injected DB sessions via `get_db`
- Wrap IO with try/except; return clear HTTP errors (`HTTPException`)
- Keep constants in `constants.py`; avoid magic strings in endpoints

## Performance & Reliability
- Cache expensive API results where possible (use `CachedMatch` tables)
- Paginate or limit result sets on read APIs
- Timeouts and basic retry for external API calls in `dota/api.py`

## Security
- Never log secrets; always mask DB URL before returning diagnostics
- CORS: only allow known origins and Vercel preview pattern

## Notes
- Keep commits small and focused; avoid changing public response shapes without coordinating the frontend
- Prefer small helpers over ad-hoc logic in endpoints

---
> Source: [johnmiko/dota](https://github.com/johnmiko/dota) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-22 -->
