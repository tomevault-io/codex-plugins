# Copilot / AI Agent Instructions — SHIELD_SAVER

Purpose: give an AI coding agent the minimal, actionable context to be productive in this repository.

1) Big-picture architecture
- Backend: Flask app in `Backend/app.py` (factory `create_app()`) — loads `config.py`, calls `init_db(app)`, and registers Blueprints for `routes/events.py`, `routes/users.py`, and `routes/ai.py`.
- Database: SQLite managed by `models/database.py` (per-request `get_db()` and `init_db()` semantics). The DB file is configured centrally (see `config.py`/`Backend/Extras/backend_notes.md`).
- Frontend: simple static `Frontend/index.html` + `assets/` (no SPA framework detected).

2) Where to make changes
- Add HTTP endpoints as Blueprints under `Backend/routes/` (follow the pattern in `events.py`, `users.py`, `ai.py`).
- Register new Blueprints in `Backend/app.py` via `app.register_blueprint(..., url_prefix="/api/yourprefix")`.
- DB access: use `get_db()` and parameterized SQL (`?` placeholders). Return rows as dicts via `sqlite3.Row`.

3) Project-specific conventions & patterns
- Blueprint + factory pattern: prefer adding feature modules as a Blueprint file under `routes/` and then registering in `app.py`.
- SQL style: raw SQL executed via the SQLite connection; use parameter substitution and commit after writes. Example: `db.execute("INSERT INTO users (name,email) VALUES (?, ?)", (name, email)); db.commit()`.
- AI integration: `routes/ai.py` currently contains placeholder endpoints (`/suggestions`, `/status`). Real ML or external model calls should be placed here (input JSON -> business logic -> JSON response).

4) Dev / run / debug commands (Windows / PowerShell)
- Run dev server (local):
  - `python Backend/app.py`
  - Server listens on `127.0.0.1:5000` by default (debug mode enabled in `if __name__ == "__main__"`).
- Example API calls (from `Backend/Extras/backend_notes.md`):
  - Create user: `curl -X POST http://127.0.0.1:5000/api/users/ -H "Content-Type: application/json" -d '{"name":"Micah","email":"micah@example.com"}'`
  - Create event: `curl -X POST http://127.0.0.1:5000/api/events/ -H "Content-Type: application/json" -d '{"title":"Hackathon","date":"2025-11-12","location":"NSU Auditorium","description":"Coding event","user_id":1}'`
  - AI suggestions: `curl -X POST http://127.0.0.1:5000/api/ai/suggestions -H "Content-Type: application/json" -d '{"preferences":["on-campus","networking"]}'`

5) Files to inspect first when working on tasks
- `Backend/app.py` — app factory & blueprint registration
- `Backend/routes/*.py` — endpoint patterns and response shapes
- `models/database.py` — `get_db()`, `init_db()` and teardown pattern
- `Backend/Extras/backend_notes.md` — quick reference with curl examples and rationale

6) Automated tests & checks
- There is no formal test harness discovered; rely on quick curl requests for endpoint verification. If adding tests, keep them small and close to the feature (e.g., unit tests for DB helpers or route handlers).

7) Integration & external dependencies
- Current code uses Flask + SQLite (see `requirements.txt` in `Backend/Extras/` for packages). External ML integrations should be added behind the AI Blueprint to keep separation of concerns.

8) Safety and non-goals for the agent
- Do not modify the database schema without also updating `init_db()` in `models/database.py` or adding migration instructions.
- Avoid changing the app factory signature — many callers expect `create_app()` to exist.

If anything in these notes is unclear or you want more emphasis on tests, CI, or deployment, tell me which section to expand or change.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mporter2024)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/mporter2024)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
