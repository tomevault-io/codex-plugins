# Project Guidelines

## Code Style

- Use TypeScript with strict types and existing path aliases: @/_ for src and @generated/_ for generated code.
- Follow existing module layout: keep schemas/types in definitions files and behavior in actions/functions/queries files.
- Keep UI components consistent with current Radix plus Tailwind patterns in src/components/ui.
- Do not hand-edit generated Prisma client files under generated/prisma.

## Architecture

- Framework: Next.js App Router with route groups in src/app/(main) and src/app/(shortener).
- API routes live in src/app/api.
- Feature logic lives in src/modules/\*.
- Database access lives in src/db/\* using Prisma and model-specific query modules.
- Shared runtime helpers (auth, cookies, jwt, logger, password, ActionResponse) live in src/lib.

## Build and Test

- Install dependencies: bun install
- Run dev server: bun run dev
- Build for production: bun run build
- Run production server: bun run start
- Lint: bun run lint
- Format: bun run format
- Database migrations: bunx prisma migrate dev
- Prisma client generation: bunx prisma generate
- Local Postgres (Docker): docker-compose -f docker/docker-compose-postgres.yaml up
- There is currently no dedicated test script in package.json.

## Conventions

- Prefer Server Actions for form workflows and return the shared ActionResponse union from src/lib/server-action.ts.
- Validate all external input with Zod before DB writes (Server Actions and API routes).
- Keep auth token handling through src/lib/cookie.ts and src/lib/jwt.ts helpers.
- Keep DB calls in src/db query modules instead of inline Prisma calls in UI components.
- Route group folders in src/app are organizational only and do not affect URL paths.

## Environment and Pitfalls

- Required environment variables: DATABASE_URL, JWT_SECRET, JWT_EXPIRATION, SALT_ROUNDS, SERVER_URL.
- Prisma schema uses a custom client output path (generated/prisma). Keep imports aligned with @generated alias.
- Logging writes to logs/info.log and logs/error.log via Winston.
- Dev and build scripts use Turbopack flags.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Blindeen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
