# Copilot / AI agent instructions — Football-Stats-App

Purpose: short, actionable guidance so an AI coding agent becomes productive quickly in this repository.

Note: this repository now contains a minimal Next.js + TypeScript scaffold. The guidance below includes project-specific references (files, scripts) to help agents get started immediately.

1) Quick discovery (project-specific)
   - Files to inspect now: `package.json`, `tsconfig.json`, `README.md`, `pages/`, `pages/api/`, `next-env.d.ts`.
   - Entry points: `pages/index.tsx` (frontend) and `pages/api/teams.ts` (example API route).
   - Read `package.json` scripts — this project uses `dev`, `build`, `start`, and `lint`.

2) Big-picture architecture (this repo)
   - This scaffold is a single-repo Next.js fullstack app (React + Next API routes). Frontend pages live under `pages/` and backend endpoints under `pages/api/`.
   - Data flow example: frontend `pages/index.tsx` fetches `GET /api/teams` to render a simple teams list. Replace API with real data sources (services or external APIs) when integrating.

3) Developer workflows (explicit commands for this scaffold)
   - Install dependencies (Windows PowerShell):

```powershell
npm ci
```

   - Run development server:

```powershell
npm run dev
```

   - Build for production / start:

```powershell
npm run build
npm run start
```

   - Linting: `npm run lint` (requires adding ESLint setup if you want strict linting rules).

4) Project-specific conventions and patterns
   - Pages & API: follow Next.js conventions — UI in `pages/*.tsx`, API routes in `pages/api/*.ts`.
   - Types: TypeScript is enabled (see `tsconfig.json`) — maintain strict types for API responses and props.
   - Adapters: when adding external integrations, prefer a `lib/` or `services/` folder at repo root for HTTP clients or DB adapters (create these folders when integrating).
   - Config: use `.env.local` for local secrets and include `.env.example` in the repo. Never commit real secrets.

5) Integration points & external dependencies
   - Look for `*.client.ts`/`*ApiClient.py`/`services/*` for third-party integrations (APIs, DB clients). Document base URLs and auth mechanisms in this file when found.
   - If `migrations/`, `alembic/`, or `prisma/` present, database schema changes require migration files and tests.

6) Making changes (rules for AI edits)
   - Prefer small, behavior-preserving commits with clear messages: `fix: <short description>` or `feat: <short description>`.
   - If package.json scripts exist, add new scripts there rather than inventing new workflow commands.
   - When adding or updating tests, run the test suite locally and include the minimal changes required to make them pass.

7) Examples & code references (repo-specific)
   - Frontend + API example: `pages/index.tsx` fetches `/api/teams`.
   - API example: `pages/api/teams.ts` returns a static list — use it as a template for adding real endpoints.
   - Scripts: `package.json` contains `dev`, `build`, `start`, and `lint`.

8) Safety & secrets
   - Never attempt to fetch secrets or external credentials. If `.env` exists, prefer `.env.example` to learn variable names.

9) When you finish a change
   - Run the project's tests and linter if present. Add a brief note here describing which files/scripts you ran.
   - Open a PR with a description that references relevant issue/ticket numbers if available.

If anything above is unclear or you want me to tailor these instructions further, tell me which framework or data sources you want (e.g., Postgres, external football API, Firebase) and I'll add concrete adapter examples, tests, and CI workflow.

---
> Source: [cedric1jpg/Footballs-stats-app](https://github.com/cedric1jpg/Footballs-stats-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-25 -->
