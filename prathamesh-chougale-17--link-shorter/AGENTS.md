This repository implements a URL shortener with a Go backend and a React (Vite + TypeScript) frontend. The goal of this file is to give focused, actionable guidance to automated coding agents so they can be immediately productive.

Key context (big picture)
- Backend (server/) — Go 1.23. Clean architecture: handlers -> service -> repository -> MongoDB. Main entrypoint: `server/cmd/main.go`.
- Frontend (landing/) — React 19 with TanStack Router, TanStack Query, TanStack Form and shadcn/ui components. Main page: `landing/src/routes/index.tsx` and root layout `landing/src/routes/__root.tsx`.
- Data flows: form submit (TanStack Form) -> useMutation (TanStack Query) -> POST `/api/links` -> repository inserts into MongoDB. Redirect flow: GET `/{code}` -> handler finds link by `short_code` and issues redirect; click count updated asynchronously via `IncrementClicks`.

Primary developer workflows (what to run)
- Start MongoDB (recommended): `docker-compose up -d` (file at repo root).
- Run backend (development):
  - cd `server` && `go run cmd/main.go`
  - Build: `cd server && go build -o link-shortener cmd/main.go`
- Run frontend (development):
  - cd `landing` && `pnpm install` (once)
  - cd `landing` && `pnpm dev` (starts Vite on 3000)
- Quick start script: `.\start.ps1` from repo root will attempt to start backend and frontend.

Project-specific patterns & conventions (what to follow)
- Layered Go project: keep business rules inside `internal/service`, DB operations in `internal/repository`, HTTP bindings in `internal/handlers`.
  - Example: `internal/service/link_service.go` contains short-code generation logic and delegates persistence to `repository`.
- Use contexts carefully: request contexts are canceled on handler return — background work (like click increments) must use a separate `context.Background()` with a short timeout (see `internal/handlers/link_handler.go`).
- MongoDB usage: repository functions return typed models from `internal/models/link.go`. Indexes are created in `internal/database/mongodb.go` (short_code unique index).
- Frontend uses TanStack Form with `createFormHook` and a shared `form-context` — field components expect `useFieldContext()` and `useStore(field.store, selector)` to select meta/errors.
  - Example field components: `landing/src/components/FormComponents.tsx`.
- Frontend data flow: useQuery for `['links']` and `['stats']`, useMutation to create/delete links; successful mutations call `queryClient.invalidateQueries({ queryKey })`.

Where to make changes (touchpoints)
- New API endpoints: edit handlers in `server/internal/handlers/*.go` and implement logic in `internal/service/*.go`.
- DB schema/evolution: change models in `server/internal/models/link.go` and update repository/index creation in `server/internal/database/mongodb.go`.
- Frontend UI & forms: edit `landing/src/routes/index.tsx`, `landing/src/components/*`, or add new route files under `landing/src/routes/` (file-based routing).
- Devtools: TanStack Query devtools are wired in `landing/src/routes/__root.tsx`.

Common pitfalls & gotchas (explicit)
- Do not use `r.Context()` in goroutines that run after the handler returns — request context will be canceled. Use a background context with timeout for background DB updates (clicks). See `link_handler.go`.
- Ensure `.env` values are loaded in `server` (we use `github.com/joho/godotenv` in `internal/config/config.go`). The repo contains `.env.example` files; local dev should copy them to `.env` or rely on environment variables.
- When adding new frontend dependencies, run `pnpm install` in `landing/` and restart Vite.
- Keep `queryKey` naming consistent (`['links']`, `['stats']`) so invalidation works properly.

Testing & verification hints
- To verify click increments: create a link via POST `/api/links` (or UI), then visit `http://localhost:8080/{shortCode}` and query `GET /api/links` to confirm `clicks` increments. If not incrementing, check that asynchronous increment goroutine used a non-cancelable context.
- Use `mongosh` or MongoDB Compass to inspect `link_shortener.links` collection directly.

Files to read first (high signal)
- `server/cmd/main.go` — wiring, CORS, route registration
- `server/internal/handlers/link_handler.go` — HTTP surface
- `server/internal/service/link_service.go` — business logic
- `server/internal/repository/link_repository.go` — DB operations
- `landing/src/routes/index.tsx` — form + queries + UI composition
- `landing/src/hooks/form.ts` & `landing/src/hooks/form-context.ts` — form wiring
- `landing/src/lib/api.ts` — client API boundaries

Editing guidance for agents
- Keep changes small and localized; follow the existing layering. When adding new endpoints, add handler → service → repository changes, and update `server/cmd/main.go` routes.
- For frontend changes, prefer adding new route files under `landing/src/routes` to follow file-based routing. Use existing UI components under `landing/src/components` and `landing/src/components/ui` for consistency.
- Document any schema changes in `server/internal/models/link.go` and update `internal/database/mongodb.go` index creation if necessary.

If you modify runtime wiring (QueryClientProvider, env handling, or server port), update `README.md`, `docs/DEVELOPMENT.md` and `start.ps1` to keep quick-start accurate.

If something is ambiguous, ask: provide the file you want changed, the desired behavior, and any example input/expected output.

-- End of file

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Prathamesh-chougale-17) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
