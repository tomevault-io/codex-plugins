# Copilot instructions (project-specific)

## Project shape
- This repo is a Prisma+TypeScript package. The Prisma schema is in `prisma/schema.prisma`.
- Prisma CLI is configured via `prisma.config.ts` (uses `defineConfig` and loads `.env` via `import "dotenv/config"`).
- The Prisma Client is generated under `src/generated/prisma` (see `generator client { output = "../src/generated/prisma" }`).
- Migrations live at `prisma/migrations` (configured in `prisma.config.ts`).

## Environment and runtime constraints
- Node is ESM (`"type": "module"` in `package.json`); prefer `import`/`export` syntax.
- TypeScript is `strict` and uses `moduleResolution: "bundler"` (see `tsconfig.json`).
- `prisma.config.ts` is included in `tsconfig.json` so Node globals (like `process`) resolve correctly in editors.
- SQLite is used; connection comes from `.env` (`DATABASE_URL="file:./dev.db"`).
  - Keep datasource URL logic in `prisma.config.ts` (schema datasource does not hardcode `url`).

## Common workflows (commands)
- Install deps: `npm install`
- Type-check (project mode): `npm run typecheck`
- Validate/format schema: `npx prisma validate` and `npx prisma format`
- Generate client (writes to `src/generated/prisma`): `npx prisma generate`
- Create/apply migrations in dev: `npx prisma migrate dev --name <migration_name>`
- Explore data: `npx prisma studio`

## TypeScript gotcha
- Do not run `npx tsc <single-file>.ts` in this repo; it may ignore `tsconfig.json` and default to a too-old target, causing errors in Prisma `.d.ts` (e.g. TS18028 about `#private`). Use `npm run typecheck` / `npx tsc -p tsconfig.json` instead.

## Conventions for changes
- When editing data models, change only `prisma/schema.prisma`, then run `npx prisma format` and `npx prisma generate`.
- Do not hand-edit anything under `src/generated/prisma/` (it is generated output).
- If you change config paths (schema, migrations, output), update `prisma.config.ts` and keep `.env` loading intact.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kksolanki1503) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
