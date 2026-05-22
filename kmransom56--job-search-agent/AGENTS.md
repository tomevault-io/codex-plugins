# Copilot Instructions for `job-search-agent`

## Build, test, and lint commands

### Local Python environment

Use `uv` for local setup:

```bash
uv venv
source .venv/bin/activate
uv pip install -r requirements.txt
```

Install Playwright browsers if you are working on Playwright-backed job sources:

```bash
playwright install chromium
```

### Frontend install / build

The frontend lives in `frontend/` and uses Vite + TypeScript:

```bash
cd frontend
npm install
npm run dev
npm run build
npm run preview
```

`npm run dev` starts Vite on port `5173` and proxies `/api` to `http://localhost:8000`.

### Run the app locally

Start Redis first, then run FastAPI and the React frontend separately:

```bash
docker run -d -p 6379:6379 redis:7-alpine
REDIS_HOST=localhost REDIS_PORT=6379 uvicorn backend.app:app --reload --port 8000
cd frontend && npm install && npm run dev
```

### Docker build / stack

```bash
docker compose up --build
curl http://localhost:11050/health
```

The main `docker-compose.yml` maps:
- API container port `8000` to host `11050`
- Frontend container port `80` to host `11056`
- Redis container port `6379` to host `6380`

### Tests

Run the main test suite:

```bash
pytest tests/ -v --cov=backend --cov-report=term-missing
```

Run a single file:

```bash
pytest tests/test_matcher.py -v
pytest tests/test_store.py -v
```

Run a single test:

```bash
pytest tests/test_matcher.py::test_score_job -v
pytest tests/test_store.py::test_save_job -v
```

`tests/test_store.py` expects a Redis instance on `localhost:6379` and uses Redis DB `15`.

There are also top-level ad hoc validation scripts (for example `test-backend-optimizations.py`, `test-google-jobs.py`, and `scripts/test_api_providers.py`) that are useful for debugging specific integrations but are separate from the main `tests/` pytest suite.

### Lint / formatting

Mirror the GitHub Actions workflow:

```bash
flake8 backend/ --count --select=E9,F63,F7,F82 --show-source --statistics
flake8 backend/ --count --exit-zero --max-complexity=10 --max-line-length=127 --statistics
black --check backend/ tests/
```

## High-level architecture

- `frontend/` is the active UI: a React + Vite + TypeScript + Tailwind CSS v4 SPA. `src/App.tsx` owns the top-level layout (sidebar, hero, tabs) and coordinates loading/refetching with `useState` and `useEffect`; there is no router or external state management layer.
- `frontend/src/api.ts` is the frontend integration boundary. Components do not build fetch requests ad hoc; they should call the wrapper functions there so request paths, error handling, and `VITE_API_URL`/`/api` behavior stay consistent.
- `frontend/src/types.ts` mirrors backend response shapes. When backend payloads change, update the TypeScript interfaces there before changing UI components.
- In development, `frontend/vite.config.ts` proxies `/api` to `http://localhost:8000`. In production, `frontend/nginx.conf` serves the SPA and proxies `/api/` to the FastAPI container named `backend`.
- `backend/app.py` is the orchestration layer. It wires together `RedisStore`, `SessionManager`, `ResumeJobMatcher`, `compute_priority`, and the source aggregation helpers. Search requests go through `fetch_all_jobs_batched()` by default; `/api/search/stream` exposes the same source pipeline as Server-Sent Events.
- `backend/sources/aggregate.py` is the source registry. Source modules are imported through `_safe_import_source()`, so broken imports degrade to partial availability instead of crashing the app. Fast sources are prioritized first, and the remaining sources run in batches.
- `backend/models.py` defines the normalized data contract: every scraper returns a `Job`, while Redis persists a `JobEntry` wrapper that adds tags, snooze/archive state, priority, and timestamps.
- `backend/store.py` is both the persistent job store and the per-source search cache. It keeps saved jobs in a Redis hash called `jobs`, deduplicated by a normalized SHA-1 of `title|company|url`, and caches search results separately under `cache:search:*`.
- Resume scoring is session-based. `POST /api/resume/upload` stores raw resume text in Redis via `SessionManager`, then rescoring walks all saved jobs and updates their `score`.
- Priority ranking is separate from match scoring. `backend/matcher.py` computes TF-IDF cosine similarity, while `backend/prioritizer.py` adds tag bonuses, preferred-source boosts, and a recency bonus when `/api/jobs` sorts results.
- MCP automation is optional and layered on top of the search app. `backend/mcp_integration.py` catalogs jobs for later application and talks to the separate `mcp-server/` stack. Core search and tagging flows still work without that service.

## Key conventions

- Frontend behavior is intentionally simple: `App.tsx` passes state and callbacks down into `Sidebar`, `Hero`, `JobsView`, `JobCard`, and `TaggedView`. Prefer extending the existing prop-driven structure over introducing routing, context, or a client-side state library unless the architecture is intentionally changing.
- Keep backend API usage centralized in `frontend/src/api.ts`. If a new endpoint is added, add a typed wrapper there and consume it from components rather than calling `fetch()` directly in component files.
- `frontend/src/types.ts` should stay aligned with `backend/models.py` and the JSON returned by `backend/app.py`; mismatches here tend to break multiple components at once because the same `JobEntry` shape flows through jobs, tagged view, bulk apply, and stats refresh.
- The frontend expects the backend base URL to differ by environment: local dev uses Vite proxying to `localhost:8000`, while production uses Nginx proxying inside the frontend container. Preserve that split when changing API paths or ports.
- New source modules should follow the existing scraper contract: expose `async def fetch(keywords: str) -> List[Job]`, normalize results into `backend.models.Job`, and return an empty list on failure instead of raising through the whole aggregation pass.
- Register new sources in `backend/sources/aggregate.py` with `_safe_import_source(...)`; that file is the canonical registry and priority/batching configuration.
- Do not assume `job.data.id` is the Redis storage key. Redis persistence is keyed by the normalized hash from `_normalize_key(...)`; mutation helpers such as `tag_job`, `archive_job`, `snooze_job`, and `update_job_score` resolve the Redis key by scanning stored `JobEntry` records.
- The search path is intentionally optimized around batching and partial progress. If you change search behavior, preserve the distinction between `/api/search` (batched full response) and `/api/search/stream` (incremental SSE events).
- Resume handling is split by use case: `/api/resume/upload` stores plain text for scoring, while `/api/resume/upload-apply` writes a PDF/DOCX resume into the shared resumes directory used by the MCP application flow.
- Preferred-source ranking is URL-based, not source-name-based: `compute_priority()` checks whether entries in `PREFERRED_SOURCES` appear in `entry.data.url`.
- The main app and the optional MCP stack use different compose files and different persistence assumptions. If you touch `/api/mcp/*` or `/api/apply`, also check `mcp-server/README.md` and `mcp-server/docker-compose.yml` because those flows depend on the external MCP service and shared `/data/resumes`.

---
> Source: [kmransom56/job-search-agent](https://github.com/kmransom56/job-search-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-22 -->
