## Purpose
Short, actionable guidance for AI coding agents working on the housegur-api FastAPI project.

Keep instructions concise and focused on discoverable repository patterns. Refer to the files named below when you need concrete examples.

## Big picture (what this app is)
- Framework: FastAPI application (entry: `app/main.py`).
- Routing: Routers live under `app/routers/` and are included in `app/main.py` with prefixes: `/auth`, `/properties`, `/transactions`.
- DB layer: SQLAlchemy engine in `app/database.py` but code uses raw SQL and stored procedures (no ORM models).
- Business logic / SQL: `app/crud.py` contains DB calls (uses `sqlalchemy.text`) and calls stored procedures (e.g., `sp_comprar_tokens`, `sp_vender_tokens`) in the `housegur` schema.
- Schemas: Pydantic request/response models are in `app/schemas.py` and used as `response_model` in routers.

## Key conventions & patterns to follow
- Use `SessionLocal()` from `app/database.py` for DB access and always close the session in a `finally` block. See `app/crud.py` and `app/routers/properties.py`.
- Prefer raw SQL via `text(...)` and stored-procedure calls for write operations (this project relies on DB side logic). Example: `app/crud.comprar_tokens` calls `CALL housegur.sp_comprar_tokens(...)`.
- Router → CRUD separation: router functions (in `app/routers/*`) are thin and delegate to `app/crud.py`. When adding features, add the API signature in `app/schemas.py`, implement DB logic in `app/crud.py`, then wire a new router and include it in `app/main.py`.
- Response shaping: Router endpoints return plain dicts or values from `app/crud.py` that must match the Pydantic response model field names (note Spanish names like `propiedad_id`, `usuario_id`, `nombre`).
- Environment: `app/database.py` reads `DATABASE_URL` via `python-dotenv`. Ensure `.env` has `DATABASE_URL` for local runs.

## When adding a new endpoint (concrete checklist)
1. Add Pydantic request/response models in `app/schemas.py`.
2. Implement DB logic in `app/crud.py` using `SessionLocal()` and `sqlalchemy.text` (or call an existing stored proc).
3. Create a router under `app/routers/` with appropriate prefix and `response_model` and delegate to `app.crud`.
4. Include the router in `app/main.py` with a clear `prefix` and `tags` entry.
5. Keep naming consistent (Spanish field names are used across the codebase).

## Files to inspect for examples
- `app/main.py` — how routers are included.
- `app/routers/auth.py` — simple POST login wiring to `crud.login_user`.
- `app/crud.py` — examples of INSERT, SELECT and stored-proc usage.
- `app/database.py` — engine/session setup and `.env` usage.
- `app/schemas.py` — Pydantic request/response shapes.

## Caveats & quick warnings
- Don't assume ORM models: adding ORM models will require adjusting conventions — prefer to follow existing raw-SQL style unless you intentionally migrate.
- DB side logic (stored procedures) is authoritative for token buy/sell flows; avoid duplicating business rules in Python.
- Consistency: field names and messages use Spanish; keep that consistent in new endpoints and schemas.

If any section is unclear or you want deeper details (examples of stored-proc inputs, a recommended local DB fixture, or automated tests), tell me which area and I'll expand the guidance.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Pablo4costa)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Pablo4costa)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
