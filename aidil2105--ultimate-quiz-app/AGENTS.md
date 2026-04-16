# Ultimate Quiz App Repository Guidance

## Project overview

Ultimate Quiz App is a solo-built full-stack study platform. This repository contains:

- React and TypeScript frontend code in `frontend/`
- FastAPI backend code in `apps/backend/`
- Supabase and PostgreSQL assets in `database/` and `migrations/`
- Deployment configuration in `deployment/`, `deploy/`, and root infra files
- Project documentation in `docs/`

## Repository conventions

- Use `Ultimate Quiz App` as the canonical project name.
- Treat `QuizGenie` as a legacy internal name that may still appear in older files.
- Keep README and setup instructions aligned with the actual repo layout.
- Do not add placeholder team wording to a solo-maintained repository.
- Do not commit real credentials, private user data, production-only identifiers, or copied logs.

## Setup expectations

- Backend local configuration is loaded from the repo-root `.env` file.
- Frontend local configuration is loaded from `frontend/.env.local`.
- Local development defaults to `http://localhost:8000` for the backend and `http://localhost:5173` for the frontend.
- `SUPABASE_SERVICE_ROLE_KEY` is optional for basic local startup, but some backend-admin flows expect it.

## High-value paths

- `README.md`
- `config/.env.example`
- `frontend/.env.example`
- `frontend/src/`
- `apps/backend/src/`
- `docs/getting-started/start-here.md`
- `docs/architecture/SYSTEM_ARCHITECTURE.md`

## Contribution expectations

- Keep changes focused and reviewable.
- Update docs when setup, behavior, or architecture changes.
- Prefer durable documentation over one-off status notes in the repo root.
- Preserve technical accuracy and avoid marketing-style claims that are not supported by the codebase.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aidil2105) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
