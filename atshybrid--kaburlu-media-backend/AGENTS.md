# Copilot instructions (Kaburlu Media Backend)

## Big picture
- Node/Express + TypeScript service. Entry: `src/index.ts` (startup + Prisma connect + core seed) → `src/app.ts` (Express middleware + route mounts).
- Primary data store is Postgres via Prisma (`src/lib/prisma.ts`, schema in `prisma/schema.prisma`).
- API docs are generated from JSDoc Swagger annotations across `src/api/**/*.ts` and served at `/api/docs` and `/api/v1/docs`.

## Dev workflows
- Dev server: `npm run dev` (alias of `npm run start:dev`, uses `ts-node-dev`)
- Build/run prod-style locally: `npm run build` then `npm run start`
- Prisma:
  - Generate client: `npm run prisma:generate` (also runs on `postinstall`)
  - Migrations: `npm run prisma:migrate:dev` (local) / `npm run prisma:migrate:deploy` (deploy)
  - Seed: `npm run seed` (or one-off scripts under `scripts/`)
- Local Postgres fallback: `docker compose up -d` + set `DATABASE_URL=postgresql://postgres:postgres@localhost:5432/kaburlu_local` (see `README.md`).

## Runtime toggles & conventions
- Env loading/validation lives in `src/config/env.ts` (imports `dotenv/config`). Prefer adding new env vars there.
- Startup is resilient to DB flakiness:
  - `DB_CONNECT_RETRIES`, `DB_CONNECT_BACKOFF_MS`
  - `ALLOW_START_WITHOUT_DB=true` to boot API even if DB is down (limited functionality)
- Prisma can be pointed at a fallback URL without changing code:
  - `DATABASE_URL_FALLBACK` + `PRISMA_PREFER_FALLBACK=true`

## Routing patterns
- Routes are mounted twice for backward compatibility: legacy root paths (e.g. `/shortnews`) and versioned `/api/v1/...`. Prefer adding new endpoints under the existing routers and ensure `/api/v1` mount works.
- Global error handling is centralized in `src/app.ts` (JSON parse errors return 400; everything else flows to the 4-arg error handler).

## Auth & principals
- Auth uses Passport JWT (`src/api/auth/jwt.strategy.ts`). `req.user` can be:
  - `kind: 'user'` (normal user principal)
  - `kind: 'device'` when token payload has `subType='device'`
- When you add protected endpoints, follow the existing pattern: `passport.authenticate('jwt', { session: false })`.

## Multi-tenancy (domain-based)
- Multi-tenancy is opt-in via `MULTI_TENANCY=true`.
- Tenant context is resolved by `src/middleware/tenantResolver.ts` using (in order):
  - `X-Tenant-Domain` header (useful for local testing)
  - `Host` / `X-Forwarded-Host`
  - `localhost` dev fallback (uses most recent active domain)
- Resolved objects are stored in `res.locals.tenant` and `res.locals.domain` (see `src/api/public/public.routes.ts`).

## AI features (prompts + providers)
- AI provider selection is key-based: Gemini if `GEMINI_API_KEY`/`GOOGLE_GENAI_API_KEY` present, else OpenAI if `OPENAI_API_KEY` present (see `src/lib/aiProvider.ts`, `src/lib/aiConfig.ts`).
- Prompt templates can come from env or DB-backed `Prompt` rows (see `prisma/schema.prisma` and `README.md` “Prompts API”).
- AI rewrite jobs use BullMQ when `REDIS_URL` is set; otherwise controllers fall back to in-process “fire-and-forget” execution (see `src/queues/aiRewrite.ts` and `src/api/articles/articles.ai.controller.ts`).
- Worker:
  - Build first, then run: `npm run jobs:bullmq` (starts `dist/workers/aiRewrite.worker.js`)
  - Dev worker alternative: `npm run jobs:ai-queue` (ts-node worker)

## Media / uploads
- Media uploads store to Cloudflare R2 (S3-compatible) using `src/lib/r2.ts` and `src/api/media/media.routes.ts`.
- R2 required env: `R2_ACCOUNT_ID`, `R2_ACCESS_KEY_ID`, `R2_SECRET_ACCESS_KEY`, `R2_BUCKET` (optionally `R2_PUBLIC_BASE_URL`, `R2_ENDPOINT`).
- Upload pipeline may transcode:
  - images → WebP (PNG preserved)
  - videos → WebM via `ffmpeg-static`/`ffprobe-static` (works on Windows too)

## When changing data models
- Update `prisma/schema.prisma`, run migration, and regenerate Prisma client.
- If a change affects “core bootstrap” behavior (roles/languages/states), update `src/lib/bootstrap.ts`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/atshybrid) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
