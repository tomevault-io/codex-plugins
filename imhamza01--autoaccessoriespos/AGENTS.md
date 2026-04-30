# Copilot / AI agent instructions for AutoAccessoriesPOS

Purpose: give AI coding agents the minimum, focused context to be productive working on this repository.

- **Big picture:** Backend is a FastAPI app at src/backend/main.py serving the SPA in `src/frontend` (mounted as static files). The app uses a custom SQLite-based `DatabaseManager` (src/backend/core/database.py) that auto-creates the DB and tables on startup and stores files under the OS app-data path (Windows: `%APPDATA%/AutoAccessoriesPOS/database/pos_main.db`). Desktop packaging lives under `src/desktop` and packaging scripts under `scripts/`.

- **Run / dev commands (Windows PowerShell):**
  - From project root run backend (with docs):
    ```powershell
    Set-Item Env:ENV development
    ## Copilot / AI agent instructions for AutoAccessoriesPOS

    Purpose: essential, repo-specific notes to get an AI coding agent productive quickly.

- **Key files to inspect:**
  - `src/backend/main.py` — FastAPI app, router registration, static mount.
  - `src/backend/core/database.py` — single authoritative DatabaseManager (auto-creates 52 tables, backup/restore, connection pooling). Avoid changing DB layout without running migrations.
  - `src/backend/api/*.py` — REST endpoints (auth, products, sales, pos, etc.). See `src/backend/api/auth.py` for patterns (dependency-injected `get_current_user`, permission checks).
  - `src/frontend` — static frontend served by FastAPI; editing here affects production UI directly when backend is serving files.
  - `scripts/` and `src/desktop` — packaging and installer flows (portable build, NSIS scripts).

- **Project-specific patterns & conventions:**
  - Database access uses `get_database_manager()` and `db_manager.get_cursor()` context manager — prefer these rather than raw sqlite3 connections.
  - Logging and audit: use `core.logger.audit_log` or `setup_logging()` in `core/logger.py` to produce consistent logs.
  - Roles follow localized identifiers (e.g. 'malik', 'munshi'); role config lives in `src/backend/core/auth.py` — keep role names stable to avoid breaking permission checks.

- **Integrations & external dependencies:**
  - Packaging relies on pywebview / pywin32 for Windows desktop builds; see `requirements.txt` for versions to match.

- **Testing / debugging hints:**
  - There are no explicit test runners in the repo root; run modules directly for integration checks. To debug endpoints, start the backend with `ENV=development` and use `/docs` or an API client.

- **What to avoid / warnings for an AI agent:**
  - Changes to `src/frontend` may be immediately served by the backend; ensure UI and API contract compatibility.

  - Add an auth-protected endpoint:
    - follow `src/backend/api/auth.py` pattern, use `APIRouter`, `Depends(get_current_user)` and `Depends(require_permission(...))` where appropriate.
    - `from core.database import get_database_manager`
    - `db = get_database_manager(); with db.get_cursor() as cur: cur.execute(...); result = cur.fetchall()`

If anything here is unclear or you'd like specific examples (more file links, sample PR templates, or rules for commit messages), tell me which area to expand. 

    If anything here is unclear or you'd like specific examples (more file links, sample PR templates, or rules for commit messages), tell me which area to expand.
    ````
    - follow `src/backend/api/auth.py` pattern, use `APIRouter`, `Depends(get_current_user)` and `Depends(require_permission(...))` where appropriate.

---
> Source: [Imhamza01/AutoAccessoriesPOS](https://github.com/Imhamza01/AutoAccessoriesPOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-30 -->
