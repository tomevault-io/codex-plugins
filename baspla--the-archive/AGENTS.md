<!-- .github/copilot-instructions.md - guidance for AI coding agents -->

This repository is a Next.js (app router) project with tRPC on the server, Drizzle ORM (SQLite) for persistence, and NextAuth for authentication. The file below provides focused, actionable guidance to help an AI code assistant be productive quickly.

1. Big picture
- Framework: Next.js (app directory). Entry pages live under `src/app/` and logged-in pages are under `src/app/(logged in)/`.
- API layer: tRPC routers in `src/trpc/routers/*`. The top-level router is `src/trpc/routers/_app.ts` and is mounted by the server caller in `src/trpc/server.tsx`.
- DB: Drizzle ORM with schema in `src/lib/db/schema.ts`, migrations in the `drizzle/` folder, and an on-disk SQLite DB at `database/archive.db`.
- Auth: NextAuth configured in `src/auth.ts` with a Drizzle adapter and provider config in `src/auth.config.ts`.

2. Developer workflows (commands)
- Local dev: `pnpm dev` (script `dev` in `package.json`, runs `next dev --turbopack`).
- Build: `pnpm build` (`next build --turbopack`).
- Start prod: `pnpm start`.
- Migrations: `pnpm migrate` runs `drizzle-kit migrate`. Schema generation: `pnpm generate`.

3. Project-specific conventions and patterns
- tRPC context: `src/trpc/init.ts` exports `createTRPCContext` which calls `auth()`; use `protectedProcedure` for authenticated endpoints and `adminProcedure` for admin-only.
- Server caller / prefetching: `src/trpc/server.tsx` provides `trpc`, `caller`, and helpers `prefetch`/`HydrateClient`. Use `caller` for server-side operations in server components.
- DB definitions: use types exported from `src/lib/db/schema.ts` (e.g., `Work`, `NewWork`, `PenName`). Migrations live in `drizzle/` and are applied automatically by `src/lib/db/schema.ts` at startup. Do not manually create migration, use `pnpm generate` instead.
- Auth/session shape: NextAuth stores user id in session at `session.user.id` (see `src/auth.ts` callbacks). Many parts of the app expect `ctx.session.user.id` to exist when authenticated.

4. Integration points & side effects
- Migrations: `src/lib/db/schema.ts` will create the `database` directory and run migrations automatically when the DB doesn't exist. Avoid deleting migrations; add new SQL files to `drizzle/` and run `pnpm migrate`.
- Environment: `OIDC_*` env vars configure the OIDC provider in `src/auth.config.ts`. `DB_LOG`, `DB_MIGRATE`, and `DB_URI` influence DB migration and logging behaviour.
- tRPC + React Query: The project uses `@trpc/tanstack-react-query` with cache helpers — maintain query keys and use `trpc.*.queryOptions()` when prefetching.

5. Common edit patterns and examples
- UI/Styling: Do not overload elements with complex Tailwind classes; keep structure simple and leave fine-tuning to the user. Rely on existing shadcn/ui components (in `src/components/ui`) whenever possible instead of building from scratch.
- Add a tRPC endpoint: create a router in `src/trpc/routers/`, export it, and add it to `src/trpc/routers/_app.ts`. Use `protectedProcedure.input(z.object(...)).mutation(...)` and access DB via `ctx` where appropriate (see `src/trpc/init.ts` for context shape).
- Add a DB column/table: add to `src/lib/db/schema.ts`. IMPORTANT: Batch all schema changes first, then run `pnpm generate` and `pnpm migrate` only once at the end of the task.
- Use server-side tRPC caller: import `caller` from `src/trpc/server.tsx` in server components or API routes to run router methods without HTTP.

6. Files to read first (high value)
- `src/trpc/init.ts` — auth, context, and procedure helpers.
- `src/trpc/server.tsx` — server caller, prefetch helpers, Hydration boundary.
- `src/lib/db/schema.ts` — full DB schema, types and migration logic.
- `src/auth.ts` & `src/auth.config.ts` — NextAuth configuration and callbacks.

7. What not to change without review
- Migration history in `drizzle/` (breaking changes will affect existing DBs).
- `src/lib/db/schema.ts`'s automatic migration runner—changing it affects startup behaviour.
- Authentication callbacks and adapter wiring in `src/auth.ts`.

8. Example snippets to reference
- tRPC protected mutation pattern: `protectedProcedure.input(z.object({pennameid: z.string()})).mutation(async ({input, ctx}) => { ... })` (see `src/trpc/routers/works.ts`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Baspla)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Baspla)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
