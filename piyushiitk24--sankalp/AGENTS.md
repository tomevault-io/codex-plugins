# Copilot Instructions for SANKALP®

## Overview
SANKALP® is a Next.js (App Router) + TypeScript + Tailwind app for gamified learning (lessons, quizzes, progress, shop). The codebase separates UI (`components/*`), business logic (`actions/*`), and data (`db/*`) and supports a full guest mode alongside Clerk auth.

## Architecture & where things live
- Routing: `app/(main)/*` (product), `app/(marketing)/*` (public), `app/admin/*` (React Admin). Root layout: `app/layout.tsx`.
- Server actions: `actions/*` (e.g., `actions/challenge-progress.ts`, `actions/user-progress.ts`) mutate state and call `revalidatePath` for learn/lesson/quests/leaderboard.
- Data layer: Drizzle ORM with `db/schema.ts`, queries in `db/queries.ts` (wrapped in `cache()` and using Clerk `auth()`), client seeded via `scripts/prod.ts` (see `npm run db:prod`).
- Auth & admin: Clerk via `components/theme-aware-clerk-provider.tsx`; admin gating in `middleware.ts` using `CLERK_ADMIN_IDS` and a route matcher for `/admin`.
- Guest mode: Client-persisted user in `lib/guest-user.ts` + `hooks/use-guest-user.ts`; gate rendering with `components/guest-user-handler.tsx`. Note: `db/queries.getUserProgress()` returns null for guests; guest-only state is managed client-side via localStorage and CustomEvent("guest-user-updated").
- State: Zustand stores in `store/*` (e.g., `use-hearts-modal.ts`, `use-exit-modal.ts`, `use-practice-modal.ts`).
- Payments: Stripe logic in `lib/stripe.ts`, webhook route under `app/api/webhooks/stripe`.

## Security, ops, and conventions
- CSP and security headers are in `next.config.mjs` (update script-src/connect-src/img-src/frame-src if you add external services). API headers (CORS, Content-Range) also configured there.
- Middleware hardens requests (blocks suspicious user agents) and restricts `/admin` to IDs from `CLERK_ADMIN_IDS`.
- Rate limiting helpers live in `lib/rate-limit.ts` using Upstash Redis (env: `UPSTASH_REDIS_REST_URL`, `UPSTASH_REDIS_REST_TOKEN`). Use `withRateLimit`/`withAuthRateLimit` in API routes.
- Logging: Winston writes to `logs/combined.log`, `logs/error.log`, `logs/security.log`; rotation configured (see `logs/README.md`).
- Theming: neon purple brand `#a259ff`; theme handling via `components/theme-provider.tsx` + `next-themes`, and Clerk is themed in `components/theme-aware-clerk-provider.tsx`.

## Developer workflows
- Install: `npm install --legacy-peer-deps`
- Dev: `npm run dev`
- DB: `npm run db:push` (Drizzle push) and `npm run db:prod` (seed via `scripts/prod.ts`); optional studio: `npm run db:studio`
- Build/Start: `npm run build` / `npm start`; Lint/Format: `npm run lint`, `npm run format:fix`
- Env (see README): `DATABASE_URL`, `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`, `CLERK_SECRET_KEY`, `STRIPE_API_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, `NEXT_PUBLIC_APP_URL`, `CLERK_ADMIN_IDS`

## Patterns to follow (with examples)
- Guest vs Auth paths: In `actions/user-progress.ts`, guest branches return lightweight results for client handling (e.g., `reduceHearts`, `refillHeartsFromVideo`). Don’t persist guest state in the DB; update via `updateGuestUser()` in the client.
- Revalidation: After mutations, revalidate `"/learn"`, `"/lesson"`, `"/quests"`, `"/leaderboard"` and per-lesson paths (see `challenge-progress.ts`).
- Queries: Use `db/queries.ts` helpers; they derive the active course and challenge completion from the authenticated user’s context.
- Admin protection: Add new admin routes under `/admin/*`; the `middleware.ts` matcher and `CLERK_ADMIN_IDS` control access.
- CSP updates: If integrating a new provider (e.g., analytics or media), extend `next.config.mjs` CSP directives accordingly to avoid runtime blocks.

## Common tasks
- Add a lesson/challenge: edit `db/schema.ts` (+ seed in `scripts/prod.ts`), ensure `db/queries.ts` exposes it, then build UI in `app/(main)/lesson/*` or `app/(main)/learn/*`.
- Theme changes: tweak `tailwind.config.ts`, `app/globals.css`, and Clerk appearance in `components/theme-aware-clerk-provider.tsx`.
- Guest course selection: call `actions/user-progress.selectCourseAsGuest()` to validate, then set `activeCourseId` via `updateGuestUser()`.

## References
- Entrypoints: `app/layout.tsx`, `app/(main)/layout.tsx`
- Logic: `actions/*`, `lib/*`, `store/*`
- Data: `db/*` (schema, queries, drizzle)
- UI: `components/*`, `app/globals.css`

— For onboarding and screenshots, see `README.md` / `READMENEW.markdown`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Piyushiitk24) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
