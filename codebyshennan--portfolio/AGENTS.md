## Purpose
Provide concise, actionable guidance for AI coding agents working on this Next.js portfolio repo so they can be productive immediately.

## Big picture
- Framework: Next.js (app router in `app/`, legacy `pages/` used for some API/OG routes).
- Entry points: UI is in `app/` (root layout at `app/layout.tsx`), server-rendered components are used by default.
- API and server endpoints: `pages/api/` contains legacy API routes (e.g. `pages/api/guestbook.ts`, `pages/api/og.tsx`) and `app/api/` contains NextAuth route handlers (`app/api/auth/[...nextauth]/route.ts`).
- Data flow: Content comes from Notion (`lib/notion.ts`) and a Postgres DB (Neon/Planetscale) accessed via Kysely (`lib/planetscale.ts`). Authentication uses NextAuth (`lib/auth.ts`).

## Key developer workflows / commands
- Install: `pnpm install` (project contains `pnpm-lock.yaml`).
- Dev server: `pnpm dev` → runs `next dev` (see `package.json`).
- Build: `pnpm run build` → runs `next build --webpack` (note the explicit `--webpack` flag in scripts).
- Setup (template cleanup): `pnpm run setup` executes `node ./lib/setup.mjs`. Be careful: `lib/setup.mjs` may delete template content depending on `IS_TEMPLATE` env.

## Important environment variables (discoverable in code)
- `NOTION_API_KEY`, `NOTION_DATABASE_ID` — used by `lib/notion.ts` to fetch blog posts.
- `DATABASE_URL` — used by `lib/planetscale.ts`; code expects Neon-compatible connection strings and handles SSL if `neon.tech` is present.
- `OAUTH_CLIENT_KEY`, `OAUTH_CLIENT_SECRET` — used by `lib/auth.ts` for GitHub provider.
- `IS_TEMPLATE` — read by `lib/setup.mjs` to decide whether to delete example content.

## Conventions & patterns to follow
- App router is primary: prefer adding UI pages/components under `app/`. Use `pages/` only for API/OG or legacy needs (this repo already mixes both).
- Server vs Client: `app/` files default to server components. Check for explicit `"use client"` when adding interactive components.
- API routes: prefer `app/api/*/route.ts` for new serverless handlers where possible; existing API logic lives in `pages/api/` (do not move without updating routing logic).
- Data access: use `lib/notion.ts` for Notion content and `lib/planetscale.ts` (Kysely) for DB access. Follow existing query patterns (see `pages/api/guestbook.ts`).
- Secrets handling: rely on environment variables; do not commit secrets. `lib/setup.mjs` will rewrite `lib/info.tsx` and `app/about/page.tsx` for template installs.

## Integration points / external services
- Vercel: uses `@vercel/analytics`, `@vercel/edge-config`, and `@vercel/og` in `next.config.js` and `app/layout.tsx`.
- Notion API: `lib/notion.ts` uses `@notionhq/client` + `notion-to-md` to convert pages to markdown.
- Postgres/Neon: `lib/planetscale.ts` uses `pg` Pool + `kysely` and schema is in `schema.sql`.
- NextAuth (GitHub): `lib/auth.ts` exports handlers used in `app/api/auth/[...nextauth]/route.ts` and `pages/api/*` session checks (see `pages/api/guestbook.ts`).

## Files to look at for examples
- `app/layout.tsx` — global layout, fonts, analytics and `Sidebar` placement.
- `lib/notion.ts` — Notion query patterns and markdown conversion.
- `lib/planetscale.ts` — DB connection setup and SSL handling for Neon.
- `pages/api/guestbook.ts` — Example Kysely insert/delete with session guard.
- `lib/setup.mjs` — Template setup script that mutates repo content; run with caution.
- `next.config.js` — Security headers and Vercel edge-config usage.

## Quick tips for AI agents editing code here
- Preserve the `app/` vs `pages/` split. If you introduce new API routes, prefer `app/api/*/route.ts` unless compatibility with `pages/` APIs is required.
- When changing DB code, run a quick check of `schema.sql` and keep Kysely `Database` typings (see `lib/planetscale.ts`).
- For auth-protected API handlers, mirror `pages/api/guestbook.ts` pattern: call `const session = await auth();` and verify `session.user`.
- Do not run `pnpm run setup` in a developer environment without confirming `IS_TEMPLATE` — it may delete `content/` and `public/images/`.

## If you need to modify environment/hosting
- Local development: create a `.env` with the keys listed above (no `.env.example` in repo — README points to upstream example).
- Deployment: Vercel is expected; `next.config.js` references Vercel Edge Config. Ensure Vercel project has required environment variables and `DATABASE_URL` with SSL if using Neon.

## What I (AI) should not change automatically
- Do not alter `lib/setup.mjs` behavior without explicit human approval.
- Do not commit or generate real secret values in the repo.

---
If anything here is unclear or you'd like more examples (e.g., common refactors, preferred test harness, or a developer `Makefile`), tell me which area to expand and I'll iterate.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/codebyshennan)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/codebyshennan)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
