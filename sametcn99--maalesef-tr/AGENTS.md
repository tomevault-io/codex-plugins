# Maalesef AI Coding Instructions

You are working on **maalesef**, a platform where job applications are always rejected by a sarcastic AI (Gemini Flash). The goal is to make rejection experiences shareable and community-focused.

## Project Structure & Stack

- **Monorepo**: Managed by [Turbo](https://turbo.build/) + Bun workspaces.
- **Backend**: NestJS, TypeORM (PostgreSQL), Passport JWT, Google Generative AI (Gemini).
- **Frontend**: Next.js 16 (App Router, React 19), Radix UI Themes, Tailwind CSS v4, Framer Motion.
- **Linting**: Biome (Frontend), ESLint/Prettier (Backend).

## Monorepo Layout

- `src/apps/backend`: API, auth, domain modules, cron-based evaluations
- `src/apps/frontend`: web app, hooks, contexts, shared UI
- Root scripts orchestrate apps with Turbo (`bun run dev`, `bun run check`, etc.)

## Core Architectures & Patterns

### Backend Patterns

- **Module Structure**: Follow standard NestJS modularity (`module`, `controller`, `service`, `repository`, `dto`, `entity`).
- **Entities**: All entities must extend `AppBaseEntity` from `src/common/entities/base.entity.ts`, which provides UUID `id`, `createdAt`, and `updatedAt`.
- **AI Evaluations**: Core logic is in `ApplicationsEvaluationService`. It processes applications asynchronously via CRON jobs.
- **Rejection Philosophy**: AI responses MUST be sarcastic, polite but dismissive, and brief (2-4 sentences). Never allow an application to be accepted.
- **Security**: Use `@CurrentUser('id')` decorator for authenticated routes.
- **Migrations**: Keep `synchronize` off for production usage; generate and run TypeORM migrations for schema changes.
- **ESM Imports**: Backend runtime is ESM; include `.js` extensions in backend imports where required.

### Frontend Patterns

- **API Communication**: Use the centralized `request` helper in `src/lib/api.ts`. It handles JWT injection, 401 auto-refresh, and FormData boundaries automatically.
- **State & Logic**: Prefer custom hooks (e.g., `use-applications.ts`) for data fetching and `Context` for global state (Auth, Job Posting).
- **UI Components**: Use Radix UI Themes and Tailwind CSS v4. Animation should use Framer Motion.
- **Routing**: Use Next.js App Router conventions under `src/app`.
- **Copy Language**: User-facing text is Turkish unless a specific requirement says otherwise.

## Developer Workflows

- **Setup**: `bun install` at root.
- **Development**: `bun run dev` at root starts both apps.
- **Health Check**: `bun run check` runs lint, format, and type checking across the monorepo.
- **Database**: Managed via Docker Compose (commonly `docker compose up -d db` for local dev).
- **Deployment**: Optimized for Coolify. Use `docker-compose.yml` or Dockerfile targets `backend-runner`/`frontend-runner`.

## Coding Priorities

1. Preserve existing architecture and naming.
2. Keep changes minimal and targeted.
3. Fix root causes instead of adding ad-hoc patches.
4. Avoid introducing new dependencies unless clearly justified.
5. Update documentation when behavior or setup changes.

## Critical Conventions

- **Migrations**: Production migrations run automatically on backend startup. Use `bun run migration:generate` in backend to create new ones.
- **Environment**: Backend requires `PORT`, `DB_*`, `JWT_*`, and `GOOGLE_AI_*`. Frontend requires `NEXT_PUBLIC_API_URL` at build time.
- **Imports**: Use absolute paths (`@/components/...` in frontend, absolute paths or relative in backend). Backend ESM imports require `.js` extensions.
- **Async Logic**: Always use `async/await` over raw Promises or Observables where possible in NestJS services.
- **Error Handling**: Throw standard NestJS exceptions (e.g., `BadRequestException`) in controllers. Frontend handles these via the `request` helper's error throwing.
- **AI Prompts**: Prompts are stored in `ApplicationsEvaluationService.buildPrompt`. Do not hardcode prompt logic elsewhere.
- **Auth Guards**: Respect global auth guard setup; explicitly mark only public routes as public.
- **Validation**: Use DTO + class-validator in backend and type-safe payloads in frontend.

## Don’ts

- Don’t bypass `src/lib/api.ts` with direct `fetch` scattered across components.
- Don’t move AI decision logic outside `ApplicationsEvaluationService`.
- Don’t return acceptance outcomes for applications.
- Don’t add silent breaking API changes without updating both frontend usage and docs.

## Example File Patterns

- **Entity**: `src/apps/backend/src/modules/applications/entities/application.entity.ts`
- **Service**: `src/apps/backend/src/modules/applications/applications.service.ts`
- **API Call**: `src/apps/frontend/src/lib/api.ts`
- **Hook**: `src/apps/frontend/src/hooks/use-applications.ts`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sametcn99)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/sametcn99)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
