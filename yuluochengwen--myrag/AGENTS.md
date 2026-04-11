# Project Guidelines

## Code Style
- Backend uses Python 3.11 + FastAPI with typed config objects in `Backend/app/core/config.py`; add new config fields via `BaseModel`/`Settings` instead of scattered constants.
- Keep API routers in `Backend/app/api/*.py`, service logic in `Backend/app/services/*.py`, and request/response models in `Backend/app/models/schemas.py`.
- Follow existing error handling in routers: raise `HTTPException` for expected failures, log unexpected exceptions (`logger.error`) then return 500.
- Frontend is plain HTML/CSS/JS (no build step). Page scripts in `Frontend/js/*.js` use `DOMContentLoaded` init, `async/await`, and `fetch` + `response.ok` checks.
- When rendering user/content text in frontend, prefer existing escaping helpers/patterns (see `Frontend/js/chat.js`, `Frontend/js/knowledge-base.js`).

## Architecture
- App entrypoint is `Backend/main.py`; all routers are mounted there and static frontend is served from `/static`.
- Main backend layers: `app/api` (HTTP layer) → `app/services` (business logic) → `app/core` (config/db/dependencies) + `app/utils`.
- Data/storage boundaries:
  - MySQL: metadata and relational records.
  - ChromaDB (`VectorDB/`): embedding vectors.
  - Neo4j: knowledge graph.
  - `KnowledgeBase/`: uploaded source documents.
- Config precedence is `.env` + defaults with YAML overrides from `Backend/config.yaml`.

## Build and Test
- Local fast start (Windows): `start-fast.bat` (activates Conda env `MyRAG`, ensures `Backend/.env`, runs uvicorn).
- Local standard start (Windows): `start.bat` (dependency check/install, DB init, then `python -m uvicorn Backend.main:app`).
- Docker stack: `docker-start.bat` or `docker-compose up -d` from workspace root.
- Full test entry: `run-tests.bat` (runs `test/test_runner.py`).
- API endpoint tests require backend running first (see `test/README.md`, `test_05_api_endpoints.py`).

## Project Conventions
- Windows scripts currently hardcode Conda paths/environment (`E:\Anaconda\...`, env name `MyRAG`); preserve this behavior unless task explicitly asks to generalize.
- API prefix convention is `/api/*`; frontend scripts should prefer relative API base where possible (some files still contain absolute localhost URLs).
- Keep collection/folder naming consistent with knowledge base IDs (`kb_<id>`) across `KnowledgeBase/` and vector collections.
- For startup or test automation changes, update corresponding `.bat` scripts rather than only README notes.

## Integration Points
- Docker service names are integration anchors (`mysql`, `ollama`, `neo4j`, `backend`, `nginx`) defined in `docker-compose.yml`.
- Backend external endpoints and credentials are injected via env vars (`MYSQL_*`, `OLLAMA_BASE_URL`, `NEO4J_*`).
- Frontend real-time progress relies on WebSocket + `client_id` flow (`Frontend/js/websocket.js`, `Frontend/js/knowledge-base.js`).
- Streaming chat uses SSE endpoints in conversation APIs (see `Frontend/js/chat.js`, `Frontend/js/stream-chat-example.js`).

## Security
- Do not introduce new hardcoded secrets; use `.env`/environment variables only.
- Existing defaults include local dev passwords in scripts/config; treat them as development-only and avoid propagating to docs/examples as production defaults.
- Keep CORS as restrictive as feasible for the target environment (`Backend/config.yaml` has tighter examples than code defaults).
- Avoid adding frontend hardcoded `http://localhost:8000`/`ws://localhost:8000` in new code; prefer relative paths or `window.location`-based URLs.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yuluochengwen)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/yuluochengwen)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
