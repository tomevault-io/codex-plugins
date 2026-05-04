# Copilot instructions for Invio

## Build, test, and lint commands

Use repo root as working directory unless noted.

| Area | Command | Notes |
| --- | --- | --- |
| Frontend install | `cd frontend && bun install` | `bun.lock` is committed; prefer Bun. |
| Frontend dev | `cd frontend && bun run dev` | Vite/SvelteKit dev server. |
| Frontend build | `cd frontend && bun run build` | Copies `../VERSION` into `frontend/static/VERSION` first. |
| Frontend type check | `cd frontend && bun run check` | Runs `svelte-kit sync` + `svelte-check`. |
| Frontend lint | `cd frontend && bun run lint` | Prettier check + ESLint. |
| Backend dev | `cd backend && deno task dev` | Watches `src/app.ts`. |
| Backend run | `cd backend && deno task start` | Production-style start command. |
| Backend lint | `cd backend && deno lint` | Uses Deno lint rules from `deno.json`. |
| Backend format check | `cd backend && deno fmt --check` | Deno formatter check. |
| Docker stack | `docker compose up -d` | Runs `frontend` and `backend` services. |

Current state of tests: there are no committed backend/frontend test files yet (`*_test.ts`, `*.test.*`, `*.spec.*` not present), so there is no project test suite or single-test command in active use.

## High-level architecture

- Invio is a split app: **SvelteKit frontend** (`frontend/`) and **Hono + SQLite backend** (`backend/`), with Docker images for each and a compose file wiring frontend -> backend (`BACKEND_URL`).
- Backend entrypoint is `backend/src/app.ts`; it initializes DB, configures security/CORS headers, and mounts three route groups under `/api/v1`:
  - `authRoutes` (`/auth/login`) for JWT issuance
  - `adminRoutes` for authenticated CRUD/export/settings/user management
  - `publicRoutes` for unauthenticated share-token invoice access and public assets
- Frontend is SSR-first. `frontend/src/hooks.server.ts` reads `invio_session`, fetches `/api/v1/settings` and `/api/v1/users/me`, and stores user/localization in `event.locals` for page server loads.
- Frontend route `frontend/src/routes/api/[...path]/+server.ts` is a backend proxy. It forwards request bodies/headers to `BACKEND_URL` and injects `Authorization: Bearer <invio_session>` when missing.
- Data layer is centralized in `backend/src/database/init.ts`: opens SQLite DB, runs SQL migrations, applies idempotent schema upgrades, seeds admin user + permission matrix, manages version-based DB backups, and installs builtin HTML templates from `backend/static/templates/`.
- Invoice export flows are shared backend concerns: PDF generation (`utils/pdf.ts`, WeasyPrint-aware runtime), XML profile exports (`utils/xmlProfiles.ts`, `ubl.ts`, `facturx.ts`, `fatturapa.ts`) are exposed from admin/public invoice routes.

## Key repository conventions

- **Permission model is explicit and shared**: resources/actions are canonicalized in `backend/src/types/index.ts` (`RESOURCES`, `ACTIONS`, `RESOURCE_ACTIONS`). New protected endpoints should use `requirePermission(resource, action)` in `admin.ts` and stay aligned with that matrix.
- **Auth/session flow**: login sets `invio_session` cookie in frontend; backend expects Bearer JWT. Prefer existing helpers (`$lib/backend.ts`, `$lib/auth.ts`) instead of ad-hoc fetch/auth parsing.
- **Settings payload normalization** is intentional: backend accepts aliases (camelCase + snake_case variants like `postalCityFormat` / `postal_city_format`) and normalizes before persistence/use. Preserve this compatibility when adding settings keys.
- **Localization pipeline**: frontend translations live in `frontend/src/lib/i18n/locales/*.json`; placeholders use `{{param}}` format. Translation maintenance uses `bun run sync-keys` and `.github/scripts/translations.py` (CI also sorts locale JSON keys).
- **UI stack conventions**: frontend styling uses Tailwind CSS v4 + daisyUI configured in `frontend/src/routes/layout.css` (`@plugin "daisyui"` and custom themes). Follow existing daisyUI utility/component patterns; `.github/daisyui.instructions.md` contains project guidance for UI edits.

---
> Source: [kittendevv/Invio](https://github.com/kittendevv/Invio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-04 -->
