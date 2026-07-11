# Copilot instructions for this repo (phoebe-server)

Purpose: FastAPI service that manages per-user PHOEBE compute sessions. Each session is a separate Python worker speaking ZMQ; the API starts/stops workers and proxies commands.

## Architecture overview

- Web API: `phoebe_server.main:app` (FastAPI) with routers in `phoebe_server/api/*`.
  - Health: `api/health.py` → `/health`, `/`.
  - Auth: `api/auth.py` → `/auth/config`, `/auth/register`, `/auth/login`, `/auth/me`. Uses Pydantic BaseModel for FastAPI request/response validation only.
  - Session mgmt (prefixed `/dash`): `api/session.py` → start/end/list sessions, memory, port pool. User-scoped via `get_current_user` dependency. All endpoints require auth (including `/dash/port-status`).
  - Command proxy: `api/command.py` → `/send/{session_id}` forwards JSON to a worker; ownership check and activity tracking.
- **Concurrency model**:
  - **All route handlers are plain `def` (not `async def`)** so FastAPI runs them in a threadpool. This is critical: the ZMQ proxy and session manager calls are synchronous and would block the asyncio event loop if declared `async def`.
  - **Single uvicorn worker** (`workers=1`). Session state (`server_registry`, port pool) is kept in module-level globals; multiple uvicorn workers would each get independent copies, causing port collisions and invisible sessions. One uvicorn process with the threadpool handles concurrency fine — all heavy compute runs in ZMQ worker subprocesses.
  - The only `async` code is `lifespan()` and `periodic_cleanup()` in `main.py`, which must be async for the ASGI lifecycle and `asyncio.sleep()`.
- **Data Models Philosophy**: 
  - **Frozen dataclasses** for configuration (`config.py`) and server-side data persistence
  - **Pydantic BaseModel** for HTTP request/response validation only (`api/auth.py` etc). Rationale: FastAPI requires this for automatic schema generation and validation; Pydantic is already a FastAPI dependency
  - Decision: Keep Pydantic models at HTTP boundary; use dataclasses internally
- Authentication (`auth/`):
  - 4 modes configured in `config.toml [auth] mode`: `none`, `password`, `jwt`, `external`.
  - `auth/jwt_auth.py`: PyJWT-based `create_access_token()` / `decode_token()`.
  - `auth/passwords.py`: bcrypt-based `hash_password()` / `verify_password()`.
  - `auth/dependencies.py`: FastAPI dependency `get_current_user()` — returns `{user_id, email, full_name, role}` or `None` for none/password modes.
  - **Startup validation**: Server refuses to start if `mode=jwt` or `mode=external` and `jwt_secret_key` is empty or `"test"` (see `main.py` lifespan).
- Session Manager: `manager/session_manager.py`
  - Tracks sessions in `server_registry` with `user_id` and `full_name` per session.
  - `list_sessions(user_id=...)` filters by owner; `get_session_owner()` for ownership checks.
  - `max_workers_per_user` limit checked before spawning.
  - Spawns workers via `psutil.Popen`, waits for readiness (ping, 30s timeout).
  - Cleans up idle sessions (configurable timeout) and terminates workers robustly (terminate → wait → kill).
  - Always frees ports on shutdown, even if worker is dead.
  - `get_current_memory_usage()` does NOT reset idle timer — only actual commands reset it.
  - ZMQ resources in `_wait_for_worker_ready()` are properly cleaned up (single context, sockets closed in finally blocks).
- Worker: `worker/phoebe_worker.py`
  - ZMQ REP server bound to `tcp://127.0.0.1:{port}` for security.
  - Returns `{success: bool, result|error, traceback?}`.
  - Uses `make_json_serializable` to normalize numpy/units for JSON.
  - Handles SIGTERM/SIGINT for graceful shutdown; ZMQ socket has `LINGER=0` for clean release.
  - `cleanup()` method ensures socket and context are closed on exit.
- Proxy: `worker/proxy.py` (ZMQ REQ client) connects to `tcp://127.0.0.1:{port}` for one request-response.
  - Has configurable receive timeout (default 5 min) and send timeout (5s).
  - Properly cleans up ZMQ context and socket in `finally` block.
  - Returns `{success: false, error: "Worker timed out"}` on timeout.
- Config: `config.py` uses frozen `@dataclass` classes + `tomllib`, three-tier config discovery (editable install → venv → system/Docker).
- Database: `database.py` provides SQLite logging with WAL mode.
  - Tables: `users` (id, email, hashed_password, first_name, last_name, role), `sessions` (with user_id, full_name), `session_metrics`, `session_commands`.
  - User CRUD: `create_user()`, `get_user_by_email()`, `get_user_by_id()`.
  - Sync mode (no threading/async).
- Background tasks: FastAPI lifespan runs periodic cleanup (every 60s) to terminate idle sessions.
- Graceful shutdown: All active sessions terminated automatically on server stop.

## Run and develop

- Setup: Create venv, activate, install: `pip install -e .[dev]`.
- Start server: `phoebe-server run --port 8001` or `uvicorn phoebe_server.main:app --host 0.0.0.0 --port 8001`.
- Tests: `pytest -v`.
- Config: Edit `config.toml` for port pool range, idle timeout, logging, auth mode/JWT settings.
- Auth modes: `none` (no auth), `password` (lab-side gate only), `jwt` (server-managed users, register+login), `external` (trust upstream JWT claims).

## API workflow (happy path)

1) `GET /auth/config` → discover auth mode.
2) `POST /auth/register` / `POST /auth/login` → get JWT (jwt mode).
3) `POST /dash/start-session` (with Bearer token) → spawns worker, returns `{ session_id, port, user_id, ... }`.
4) `POST /send/{session_id}` with body `{ "command": "set_value", "twig": "period@binary", "value": 1.5 }` → ownership check, proxy to worker.
5) `POST /dash/end-session/{session_id}` → ownership check, terminates worker, frees port.
6) `GET /dash/sessions` → lists sessions owned by current user.

## Key patterns

- **Sync endpoints**: All route handlers are `def` (not `async def`). FastAPI runs them in a threadpool, which is required because ZMQ and session manager calls are blocking. Using `async def` would serialize all requests through the event loop.
- Session ownership: `_check_ownership()` in session.py raises 403 if user_id doesn't match.
- Config: frozen dataclasses, three-tier discovery, `get_args()` helper to filter TOML keys.
- Routers: group endpoints per file, include in `main.py`.
- Worker contract: every command returns JSON-serializable data; errors become `{success:false, error, traceback}`.
- **ZMQ resource management**: Always create context/socket in `try`, close in `finally`. Set `LINGER=0` on sockets. Set timeouts (`RCVTIMEO`, `SNDTIMEO`) to prevent indefinite blocking.
- CORS: wide-open in dev (`*`); configure for production.
- **JWT Security**: `jwt_secret_key` is used by PyJWT (HS256) to sign/verify tokens. MUST be:
  - Kept secret (never commit to VCS; use env vars in production)
  - Strong and random (≥32 chars; generate with `python -c "import secrets; print(secrets.token_urlsafe(32))"`)  
  - Consistent across all server instances issuing/validating tokens
  - Rotated if compromised
  - Set via env var: `export PHOEBE_JWT_SECRET_KEY="..."`; config.toml will read it
  - Server refuses to start with empty or `"test"` secret in jwt/external mode

- App: `phoebe_server/main.py` (lifespan, CORS, routers), CLI: `phoebe_server/cli.py`.
- Auth: `phoebe_server/auth/` (jwt_auth.py, passwords.py, dependencies.py), API: `phoebe_server/api/auth.py`.
- Sessions: `phoebe_server/api/session.py`, Manager: `phoebe_server/manager/session_manager.py`.
- Worker: `phoebe_server/worker/phoebe_worker.py`, Proxy: `phoebe_server/worker/proxy.py`.
- Config: `phoebe_server/config.py`, example: `config.toml`.
- Database: `phoebe_server/database.py`, default location: `data/sessions.db`.

---
> Source: [aprsa/phoebe-server](https://github.com/aprsa/phoebe-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-11 -->
