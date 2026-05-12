# ReadNews — Copilot Instructions

## Project Overview
ReadNews is a self-hosted web app that aggregates RSS/Atom feeds and delivers scheduled Telegram (and optional email) digests.

## Stack
- **Backend**: FastAPI · APScheduler · SQLAlchemy + SQLite · feedparser · BeautifulSoup4 · httpx · uvicorn
- **Frontend**: React + TypeScript · Vite
- **Deployment**: Docker Compose (`restart: unless-stopped`, `TZ=America/Bogota`). Ports: 5173 (frontend), 8000 (backend).
- **Dev machine**: Fedora at `/home/alejo/Documents/ReadNews`. Server: Windows + Docker Desktop at 192.168.1.2.

## Conventions
- All settings are persisted to SQLite via `AppSetting` table — no `.env` file needed at runtime.
- DB migrations are idempotent via `run_migrations()` in `backend/app/db.py` (ALTER TABLE with swallowed exceptions).
- Scheduler uses `_JOB_WEEKDAY` / `_JOB_WEEKEND` constants with `_safe_add_jobs()`.
- Frontend has no local `node_modules`; all builds/runs happen inside Docker. TypeScript errors in VS Code editor are false positives.
- `sudo` password on dev machine: `Fedora.pc`.

## Key Files
- `frontend/src/App.tsx` — Main React component (~1000 lines), all 16 features, sidebar layout.
- `frontend/src/index.css` — Global styles: white background, black sidebar (`#111`), purple accent `#7c3aed`.
- `backend/app/worker.py` — Digest fetch + send logic.
- `backend/app/scheduler.py` — APScheduler weekday/weekend cron jobs.
- `backend/app/routers/` — FastAPI routers: sources, settings, logs.
- `backend/app/services/` — telegram.py, email.py, opml.py.

## Development Notes
- Work through each checklist item systematically.
- Keep communication concise and focused.
- Follow development best practices.

---
> Source: [alejo9824/ReadNews](https://github.com/alejo9824/ReadNews) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-12 -->
