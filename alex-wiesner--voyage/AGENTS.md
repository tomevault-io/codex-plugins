# Voyage Development Instructions

Voyage is a self-hosted travel companion web application built with SvelteKit frontend and Django backend, deployed via Docker.

**ALWAYS follow these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**

## Pre-Release Policy
Voyage is **pre-release** — not yet in production use. During pre-release:
- Architecture-level changes are allowed, including replacing core libraries (e.g. LiteLLM).
- Prioritize correctness, simplicity, and maintainability over backward compatibility.
- Before launch, this policy must be revisited and tightened for production stability.

## Architecture

**Stack**: SvelteKit 2 (TypeScript) frontend · Django REST Framework (Python) backend · PostgreSQL + PostGIS database · Memcached · Docker · Bun (frontend package manager)

**Key architectural pattern — API Proxy**: The frontend never calls the Django backend directly. All API calls go to `src/routes/api/[...path]/+server.ts`, which proxies requests to the Django server (`http://server:8000`), injecting CSRF tokens and managing session cookies. This means frontend fetches use relative URLs like `/api/locations/`.

**AI Chat**: The AI travel chat assistant is embedded in Collections → Recommendations (component: `AITravelChat.svelte`). There is no standalone `/chat` route. Chat providers are loaded dynamically from `GET /api/chat/providers/` (backed by LiteLLM runtime list + custom entries like `opencode_zen`). Chat conversations stream via SSE through `/api/chat/conversations/`. Provider config lives in `backend/server/chat/llm_client.py` (`CHAT_PROVIDER_CONFIG`). Default AI provider/model saved via `UserAISettings` in DB (authoritative over browser localStorage). Chat composer supports per-provider model override via dropdown selector fed by `GET /api/chat/providers/{provider}/models/` (persisted in browser `localStorage` key `voyage_chat_model_prefs`). Collection chats inject collection UUID + multi-stop itinerary context; system prompt guides `get_trip_details`-first reasoning and confirms only before first `add_to_itinerary`. LiteLLM errors are mapped to sanitized user-safe messages via `_safe_error_payload()` (never exposes raw exception text). Invalid tool calls (missing required args) are detected and short-circuited with a user-visible error — not replayed into history. Chat agent tools (`get_trip_details`, `add_to_itinerary`) respect collection sharing — both owners and `shared_with` members can use them; `list_trips` remains owner-only. Tool outputs render as concise summaries (not raw JSON); `role=tool` messages are hidden from display and reconstructed on reload via `rebuildConversationMessages()`.

**Services** (docker-compose):
- `web` → SvelteKit frontend at `:8015`
- `server` → Django (via Gunicorn + Nginx) at `:8016`
- `db` → PostgreSQL + PostGIS at `:5432`
- `cache` → Memcached (internal)

**Authentication**: Session-based via `django-allauth`. CSRF tokens fetched from `/auth/csrf/` and passed as `X-CSRFToken` header on all mutating requests. There's also a `X-Session-Token` header path for mobile clients (handled in middleware).

## Codebase Conventions

**Backend layout**: The Django project lives in `backend/server/`. Apps are `adventures` (core: locations, collections, itineraries, notes, transportation), `users`, `worldtravel` (countries/regions), `integrations`, `achievements`, and `chat` (LLM chat agent with dynamic provider catalog). Views inside `adventures` are split into per-domain files under `adventures/views/` (e.g. `location_view.py`, `collection_view.py`).

**Backend patterns**:
- DRF `ModelViewSet` subclasses for all CRUD resources; custom actions with `@action`
- `get_queryset()` always filters by `user=self.request.user` to enforce ownership
- `GenericForeignKey` (`content_type` + `object_id`) used in `CollectionItem` to allow collections to contain heterogeneous items
- Background geocoding runs in a daemon thread via `threading.Thread`
- Money fields use `djmoney` (`MoneyField`); geospatial queries use PostGIS via `django-geojson`

**Frontend layout**: Source lives in `frontend/src/`. Pages are in `src/routes/` (file-based SvelteKit routing). Shared code lives in `src/lib/`:
- `src/lib/types.ts` — all TypeScript interfaces (`Location`, `Collection`, `User`, `Visit`, etc.)
- `src/lib/index.ts` — general utility functions
- `src/lib/index.server.ts` — server-only utilities (used in `+page.server.ts` and `+server.ts` files)
- `src/lib/components/` — Svelte components organized by domain (`locations/`, `collections/`, `map/`, `cards/`, `shared/`); includes `AITravelChat.svelte` for Collections chat
- `src/locales/` — i18n JSON files (uses `svelte-i18n`); wrap all user-visible strings in `$t('key')`

**Frontend patterns**:
- Use `$t('key')` from `svelte-i18n` for all user-facing strings; add new keys to locale files
- API calls always use `credentials: 'include'` and the `X-CSRFToken` header (get token from the `csrfToken` store/prop)
- Reactive state updates require reassignment to trigger Svelte reactivity: `items[i] = updated; items = items`
- DaisyUI component classes (e.g. `btn`, `card`, `modal`) plus Tailwind utilities for layout; use daisyUI semantic color names (`bg-primary`, `text-base-content`) not raw Tailwind colors so themes work
- Map features use `svelte-maplibre` with MapLibre GL; geospatial data is GeoJSON

**Running a single Django test**:
```bash
docker compose exec server python3 manage.py test adventures.tests.TestLocationAPI
```

## Working Effectively

### Essential Setup (NEVER CANCEL - Set 60+ minute timeouts)
Run these commands in order:
- `cp .env.example .env` - Copy environment configuration
- `time docker compose up -d` - **FIRST TIME: 25+ minutes, NEVER CANCEL. Set timeout to 60+ minutes. Subsequent starts: <1 second**
- Wait 30+ seconds for services to fully initialize before testing functionality

### Development Workflow Commands
**Frontend (SvelteKit — prefer Bun):**
- `cd frontend && bun install` - **45+ seconds, NEVER CANCEL. Set timeout to 60+ minutes**
- `cd frontend && bun run build` - **32 seconds, set timeout to 60 seconds**
- `cd frontend && bun run dev` - Start development server (requires backend running)
- `cd frontend && bun run format` - **6 seconds** - Fix code formatting (ALWAYS run before committing)
- `cd frontend && bun run lint` - **6 seconds** - Check code formatting
- `cd frontend && bun run check` - **12 seconds** - Run Svelte type checking (0 errors, 6 warnings expected)

**Backend (Django with Python — prefer uv for local tooling):**
- Backend development requires Docker - local Python pip install fails due to network timeouts
- `docker compose exec server python3 manage.py test` - **7 seconds** - Run tests (6/39 pre-existing failures expected; 9 chat tests all pass)
- `docker compose exec server python3 manage.py help` - View Django commands
- `docker compose exec server python3 manage.py migrate` - Run database migrations
- Use `uv` for local Python dependency/tooling commands when applicable

**Full Application:**
- Frontend runs on: http://localhost:8015
- Backend API runs on: http://localhost:8016
- Default admin credentials: admin/admin (from .env file)

## Validation

### MANDATORY End-to-End Testing
**ALWAYS manually validate any new code by running through complete user scenarios:**
1. **ALWAYS run the bootstrapping steps first** (copy .env, docker compose up)
2. **Navigate to http://localhost:8015** - Verify homepage loads correctly
3. **Test basic functionality** - Homepage should display travel companion interface
4. **CRITICAL**: Some login/navigation may fail due to frontend-backend communication issues in development Docker setup. This is expected.

### Pre-Commit Validation (ALWAYS run before committing)
**ALWAYS run these commands to ensure CI will pass:**
- `cd frontend && bun run format` - **6 seconds** - Fix formatting issues
- `cd frontend && bun run lint` - **6 seconds** - Verify formatting is correct (should pass after format)
- `cd frontend && bun run check` - **12 seconds** - Type checking (some warnings expected)
- `cd frontend && bun run build` - **32 seconds** - Verify build succeeds

## Critical Development Notes

### Configuration Issues
- **KNOWN ISSUE**: Docker development setup has frontend-backend communication problems
- The frontend may display "500: Internal Error" when navigating beyond homepage
- For working application, use production Docker setup or modify `PUBLIC_SERVER_URL` in .env
- **DO NOT attempt to fix these configuration issues** - focus on code changes only

### Docker vs Local Development
- **PRIMARY METHOD**: Use Docker for all development (`docker compose up -d`)
- **AVOID**: Local Python development (pip install fails with network timeouts)
- **AVOID**: Trying to run backend outside Docker (requires complex GDAL/PostGIS setup)

### Branch Hygiene
- **ALWAYS** commit and merge completed feature branches promptly once validation passes.
- Do not leave finished work lingering in long-lived feature branches or worktrees.

### Expected Test Failures
- Frontend check: 0 errors and 6 warnings expected (pre-existing in `CollectionRecommendationView.svelte` + `RegionCard.svelte`)
- Backend tests: 6 out of 39 Django tests fail (pre-existing: 2 user email key errors + 4 geocoding API mocks; 9 new chat tests all pass) - **DO NOT fix unrelated test failures**

### Build Timing (NEVER CANCEL)
- **Docker first startup**: 25+ minutes (image downloads)
- **Docker subsequent startups**: <1 second (images cached)
- **Frontend bun install**: 45 seconds
- **Frontend build**: 32 seconds
- **Tests and checks**: 6-12 seconds each

## Common Tasks

### Repository Structure
```
Voyage/
├── frontend/           # SvelteKit web application
│   ├── src/           # Source code
│   ├── package.json   # Node.js dependencies and scripts
│   └── static/        # Static assets
├── backend/           # Django API server
│   ├── server/        # Django project
│   ├── Dockerfile     # Backend container
│   └── requirements.txt # Python dependencies
├── docker-compose.yml # Main deployment configuration
├── .env.example       # Environment template
└── install_voyage.sh # Production installer
```

### Key Scripts and Files
- `frontend/package.json` - Contains all frontend build scripts
- `backend/server/manage.py` - Django management commands
- `docker-compose.yml` - Service definitions (frontend:8015, backend:8016, db:5432)
- `.env` - Environment configuration (copy from .env.example)

### Development vs Production
- **Development**: Use `docker compose up -d` with .env file
- **Production**: Use `./install_voyage.sh` installer script
- **CI/CD**: GitHub Actions in `.github/workflows/` handle testing and deployment

### Common Error Patterns
- **"500: Internal Error"**: Frontend-backend communication issue (expected in dev setup)
- **"Cannot connect to backend"**: Backend not started or wrong URL configuration
- **"pip install timeout"**: Network issue, use Docker instead of local Python
- **"Frontend build fails"**: Run `bun install` first, check Node.js version compatibility

## Troubleshooting Commands
```bash
# Check Docker services status
docker compose ps

# View service logs
docker compose logs web      # Frontend logs  
docker compose logs server   # Backend logs
docker compose logs db       # Database logs

# Restart specific service
docker compose restart web   # Frontend only
docker compose restart server # Backend only

# Complete restart
docker compose down && docker compose up -d
```

## Important File Locations
- Configuration: `.env` file in repository root
- Frontend source: `frontend/src/`
- Backend source: `backend/server/`
- Static assets: `frontend/static/`
- Database: Handled by Docker PostgreSQL container
- Documentation: `documentation/` folder

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Alex-Wiesner) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
