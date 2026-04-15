
# Rasaswadaya AI Agent Instructions

## Monorepo Architecture
- Uses a pnpm+Turborepo monorepo: `apps/` (web, api, mobile, ai), `packages/` (database, shared code)
- **apps/web**: Next.js 16, React 19, Tailwind, Prisma (Neon adapter), main user-facing app
- **apps/api**: NestJS 11, REST API, business logic, tests
- **packages/database**: Prisma schema, DB utilities, shared via workspace protocol

## Key Workflows
- **Install**: `pnpm install` at root (installs all apps/packages)
- **Dev server**: `pnpm run dev` (Next.js), `pnpm run dev` in `apps/api` (NestJS)
- **Build**: `pnpm run build` (Next.js), `pnpm run build` in `apps/api` (NestJS)
- **Test (API)**: `pnpm run test`, `pnpm run test:e2e`, `pnpm run test:cov` in `apps/api`
- **Lint**: `pnpm run lint` in each app

## Database & Auth
- **Prisma**: Central schema in `packages/database/prisma/schema.prisma`, shared via `@repo/database`
- **Neon/Postgres**: Uses Neon serverless adapter for Prisma, connection string via `DATABASE_URL`
- **User model**: See `schema.prisma` for roles, relations (User, Artist, Event, Store, etc.)
- **Auth**: JWT (via `jose`), session cookies, password hashing with `bcryptjs` (see `apps/web/src/app/actions/auth.ts`)

## Environment & Deployment
- **.env variables**: Must set DB URLs (see Vercel/turbo warnings for required vars)
- **Vercel**: Deploys Next.js app, uses Turborepo for build orchestration
- **Turbo**: See `turbo.json` for build/lint/dev task config

## Project Conventions
- **TypeScript everywhere**
- **Absolute imports**: Use `@/` for web app, `@repo/database` for shared DB code
- **Prisma client**: Singleton pattern in `apps/web/src/lib/db.ts`
- **Session**: JWT in cookies, 24h expiry, see `apps/web/src/lib/auth.ts`
- **API**: RESTful, NestJS modules in `apps/api/src/`
- **UI**: Next.js app directory, pages in `apps/web/src/app/`, components in `apps/web/src/components/`

## Common Issues
- **Missing deps**: Install `bcryptjs`, `jose` if build fails (see `apps/web/src/app/actions/auth.ts`)
- **Env vars**: Ensure all DB-related env vars are set in Vercel/turbo

## Examples
- **User signup**: `apps/web/src/app/actions/auth.ts` (hashes password, creates user, sets session cookie)
- **DB query**: `apps/web/src/lib/db.ts` (Prisma client usage)
- **API test**: `apps/api/test/app.e2e-spec.ts`

---
For more, see each app's README and `packages/database/prisma/schema.prisma` for data model.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rowneth) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
