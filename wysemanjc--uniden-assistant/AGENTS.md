# Copilot Instructions (Strict)

## ⚠️ CRITICAL: Script Governance (HIGHEST PRIORITY) ⚠️

The Uniden Assistant project uses **TWO AND ONLY TWO** management scripts. Adherence to this is **MANDATORY**:

### 1. setup_uniden.sh - Environment Setup (ONE-TIME + UPDATES)
- **Purpose**: Complete environment setup and initialization
- **When to use**: First-time setup, dependency updates, environment repairs
- **Features**:
  - Installs OS packages (Debian/apt-get: python3, python3-venv, python3-dev, build-essential, libssl-dev, libffi-dev, curl, git)
  - Validates Python 3.8+ and Node.js 16+ requirements
  - Creates and configures Python virtual environment
  - Installs Python dependencies from requirements.txt
  - Installs frontend dependencies via npm
  - Runs database migrations
  - Creates config.env from template if missing
  - Tests database connectivity
  - **Rerunnable and idempotent** - safe to run multiple times
- **Usage**: `./setup_uniden.sh`

### 2. uniden_assistant - Application Lifecycle Management (DAILY USE)
- **Purpose**: Start, stop, restart, monitor the application
- **Commands**:
  - `./uniden_assistant start` - Start backend and frontend
  - `./uniden_assistant stop` - Stop all services
  - `./uniden_assistant restart` - Restart all services
  - `./uniden_assistant status` - Show service status
  - `./uniden_assistant logs [backend|frontend|both]` - View logs
  - `./uniden_assistant follow [backend|frontend]` - Follow logs in real-time
  - `./uniden_assistant help` - Show help
- **Features**:
  - Kills old/stale processes before starting (no port conflicts)
  - Validates setup before starting
  - Runs database migrations and connectivity checks
  - Tracks PIDs in .pids/ directory
  - Writes logs to .logs/ directory
  - **NEVER pipes through tail or other commands**
  - Uses proper venv paths (venv/bin/python, not bare 'python')
  - Graceful shutdown with SIGTERM, fallback to SIGKILL
  - Port conflict detection and resolution
  - Comprehensive error handling and status reporting

### 🚫 FORBIDDEN ACTIONS (NEVER DO THESE):
1. **NEVER** create or suggest creating `start.sh` or `setup.sh`
2. **NEVER** create additional orchestration scripts
3. **NEVER** suggest running `python manage.py runserver` or `npm run dev` directly
4. **NEVER** manually start, stop, or restart backend or frontend services directly
5. **NEVER** pipe orchestration script through tail or head: `./uniden_assistant start | tail`
6. **NEVER** pipe any command through head or tail - these commands NEVER work correctly
7. **NEVER** use bare `python` command - always use explicit venv path: `venv/bin/python`
8. **NEVER** bypass the orchestration script for application lifecycle management
9. **NEVER** add functionality to scripts without updating this documentation
ALL application lifecycle management
3. **ALWAYS** use explicit venv Python paths: `venv/bin/python` (NEVER bare `python`)
4. **ALWAYS** keep `setup_uniden.sh` updated when adding new dependencies
5. **ALWAYS** update both scripts together when infrastructure changes
6. **ALWAYS** refer users to these scripts for setup and startup issues
7. **ALWAYS** check these scripts are executable (chmod +x)
8. **ALWAYS** use the orchestration script for debugging - check logs with `./uniden_assistant logs`
9. **ALWAYS** read full log files directly with `cat` if needed, NEVER use `head` or `tail` in commandsw dependencies
4. **ALWAYS** update both scripts together when infrastructure changes
5. **ALWAYS** refer users to these scripts for setup and startup issues
6. **ALWAYS** check these scripts are executable (chmod +x)

### Script Maintenance Rules:
- When adding Python packages: Update backend/requirements.txt AND run `./setup_uniden.sh`
- When adding OS dependencies: Update install_os_packages() in setup_uniden.sh
- When changing ports: Update BACKEND_PORT/FRONTEND_PORT in uniden_assistant script
- When adding new services: Update start/stop functions in uniden_assistant script
- Keep script documentation in sync with actual functionality

## Application Structure

The Uniden Assistant application is organized as a single Django project with multiple apps:

- **backend/uniden_assistant/** - Main Django project containing all apps
  - **hpdb/** - HPDB database API
  - **usersettings/** - User settings and favorites API
  - **uniden_manager/** - Orchestration/manager API
  - **appconfig/** - Application configuration API
  - **settings.py, urls_main.py, wsgi.py, asgi.py** - Django project configuration

## Architecture Enforcement

The application must adhere to a strict multi‑tier API hierarchy:

1) **Uniden Manager** (`backend/uniden_assistant/uniden_manager/`)
  - Base: /api/uniden_manager/
  - Front end must ONLY call this API.
  - This tier MUST NOT directly access any database.
  - It may only call the internal tier APIs (favourites).

2) **Favourites** (`backend/uniden_assistant/favourites/`)
  - Base: /api/favourites/
  - Dedicated SQLite database file for favourites and scanner profiles.
  - MUST NOT read from or write to any other database.

## Database Rules

- SQLite default database is reserved for Django core tables only (auth/admin/sessions/etc.).
- Favourites data is stored in a dedicated SQLite database configured via db_router.py.

## Development Rules

- All new front‑end API calls must use /api/uniden_manager/.
- Do not add direct database access to Uniden Manager.
- Keep data access confined to the correct tier and database.
- When creating new models, ensure they have the correct `app_label` in their Meta class.
- Import models from their respective apps (e.g., `from uniden_assistant.favourites.models import FavoritesList`).

## Configuration

- Environment variables are stored in `config.env` (not `.env`)
- Frontend environment: `frontend/.env.local`
- Do NOT create or reference `.env` or `.env.example` files

## Documentation Governance

- **Before creating any new specification docs**, check the Input_File_Specification folder for existing .md documentation and reuse it.
- Only create new spec docs if the needed information does not exist there; otherwise update or link to the existing docs.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/WysemanJC)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/WysemanJC)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
