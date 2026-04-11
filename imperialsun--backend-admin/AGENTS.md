# Copilot / AI agent instructions — Demeter Admin Panel

Quick facts

- Local dev: `npm run dev`
- Build: `npm run build`
- Lint: `npm run lint`
- Tests: `npm run test:ci`
- Docker image: `docker build .`

Big-picture architecture

- Frontend stack: React 19 + TypeScript + Vite + Tailwind CSS 4 + React Router 7.
- Entry flow:
  - `src/main.tsx` wires React Query, router and session provider.
  - `src/App.tsx` defines protected routes and admin-only screens.
  - `src/routes/*` contains page-level screens.
- Security/data layer:
  - `src/lib/admin-api.ts` is the low-level fetch client. It is responsible for `credentials: include`, JSON parsing and `X-Admin-CSRF` injection on mutations.
  - `src/lib/admin-client.ts` is the typed API surface used by the pages.
  - `src/lib/admin-security.ts` stores the admin CSRF token in memory only.
  - `src/lib/runtime-config.ts` reads `window.__APP_RUNTIME_CONFIG__` and must remain the only runtime URL source.
- Session layer:
  - `src/lib/admin-session-context.tsx` bootstraps `/admin/auth/me` then `/admin/auth/refresh`.
  - `src/lib/use-admin-session.ts` is the only hook pages/components should use for session state.
- UI layer:
  - `src/components/layout/*` holds the shell.
  - `src/components/ui/*` holds small local primitives copied/adapted into this repo.

Project-specific rules

- This repo is autonomous. Never add imports from sibling repos (`../Backend`, `../Front user`).
- All admin traffic must target `/api/v1/admin/*` and `/api/v1/admin/auth/*`.
- Never persist sensitive auth data in `localStorage`, `sessionStorage`, IndexedDB or logs.
- Keep the backend as the authorization source of truth. UI guards may hide actions but must not replace backend checks.
- Mutating requests must continue to send `X-Admin-CSRF` when available.
- Keep `runtime-config.js` externalized and non-cacheable so deployments can swap backend URL without rebuilding the image.

Data-fetching conventions

- Use React Query for server data.
- Keep mutation side effects explicit:
  - invalidate the affected query keys
  - show a concise feedback message
  - do not silently swallow failures
- Favor small, typed client helpers in `src/lib/admin-client.ts` over ad hoc `fetch()` calls inside route files.

UI conventions

- Preserve the established admin visual language: warm neutrals, dense operator-oriented tables, rounded cards.
- Keep forms explicit and operational. This app is a back-office, not a marketing site.
- Prefer accessible labels and deterministic IDs because tests rely on them.

Testing requirements

- Every new function, component, route, mutation helper or security-sensitive change must ship with tests.
- Prefer:
  - unit tests for `src/lib/*`
  - component tests for `src/routes/*`
  - route-guard tests for `src/App.tsx`
- Use Vitest + Testing Library.
- Mock backend HTTP through the client modules; do not reach real services in unit tests.

Pre-commit validation

- Run and fix all failures before asking for a commit:
  - `npm run lint`
  - `npm run test:ci`
  - `npm run build`
  - `docker build .`
- Before any commit or push, present the diff and the outputs of the commands above and wait for explicit user approval.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/imperialsun)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/imperialsun)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
