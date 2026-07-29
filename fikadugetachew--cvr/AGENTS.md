<!-- Project-specific Copilot instructions for the CVR monorepo -->
# Quick context (what this repo is)
- Monorepo managed by pnpm. Two main apps: `@cvr/api` (Express + Sequelize MySQL API) and `@cvr/web` (React + Vite frontend).
- API lives in `apps/api` and is the primary backend for auth, resident profiles, kebele operations and file uploads.

# Where to look first
- `apps/api/src/app.js` — Express app wiring (middleware: helmet, cors, morgan, rateLimit, static `/uploads`).
- `apps/api/src/server.js` — starts server and ensures DB connects via `db.sequelize.authenticate()`.
- `apps/api/src/db` — Sequelize setup: `sequelize.js` (connection), `models/index.js` (dynamic model loader), `models/*.js` (model factories that export a default function returning a Model).
- `apps/api/src/controllers` — request handlers and transaction patterns (see `auth.controller.js` for transactional signup flow).
- `apps/api/migrations` — Sequelize migrations reflect DB shape. Use `pnpm --filter @cvr/api migrate` to run.

# Important commands (run from repo root)
- Install & run both apps in dev: pnpm install then `pnpm dev` (monorepo dev script proxies to `dev:web` and `dev:api`).
- Run only API dev: `pnpm dev:api` (runs `nodemon` in `@cvr/api`).
- Start API production-like: `pnpm --filter @cvr/api start` (note: uses Node ESM specifier resolution flag).
- Migrations & seeds (API): `pnpm --filter @cvr/api migrate` and `pnpm --filter @cvr/api seed`.
- Tests (API): `pnpm test` or `pnpm --filter @cvr/api test` (Jest with experimental vm modules flag).

# Patterns and conventions Copilot should follow
- ESM modules everywhere (package.json has "type": "module"). Use import/export, not require/module.exports.
- Sequelize model files export a default factory: default(sequelize) => Model. Models are registered dynamically in `models/index.js`.
- Table/column naming: migrations use underscored column names (snake_case) and models set `underscored: true` — follow snake_case for DB fields.
- UUIDs are used for primary keys (see `user` migration and model). When creating fixtures or tests, use uuidv4 or Sequelize.literal('(UUID())') as appropriate.
- Transactions: controllers often use transactions (see `signupResident` in `auth.controller.js`). Wrap multiple DB writes in a transaction and commit/rollback correctly.
- File uploads: multer stores files under `apps/api/uploads`; controllers record forward-slash paths (replace backslashes with `/`).
- Token handling: access tokens are JWTs (`services/token.service.js`), refresh tokens are high-entropy strings hashed in DB. Do NOT store plain refresh tokens in DB.

# Code examples to mirror
- Create a Sequelize model factory (pattern):
  - export default (sequelize) => { class X extends Model { static associate(models) { /* ... */ } } X.init({...}, { sequelize, modelName: 'X', underscored: true }); return X; }
- Use transactions in controllers:
  - const t = await db.sequelize.transaction(); try { await Model.create(..., { transaction: t }); await t.commit(); } catch (err) { await t.rollback(); throw err; }
- Lookup & rotate refresh token (use `sha256` hash helper and `RefreshToken` model as shown in `services/token.service.js`).

# Tests and linting
- Tests: API uses Jest. Run `pnpm --filter @cvr/api test`. Tests run with `--runInBand` and require Node flag `--experimental-vm-modules` (already set in package.json test script).
- Lint: `pnpm lint` at repo root runs ESLint across the monorepo. Pre-commit hooks via husky and lint-staged are configured.

# External integrations & env
- MySQL database configured via environment variables (DB_NAME, DB_USER, DB_PASS, DB_HOST, DB_PORT). See `apps/api/src/db/sequelize.js`.
- JWT secret and token expiry via env: JWT_SECRET, ACCESS_TOKEN_EXPIRES_IN, REFRESH_TOKEN_DAYS.
- File upload storage is local `uploads/` directory. In production, replace or adapt storage and path handling consistently.

# Small gotchas for Copilot suggestions
- Keep ESM import style and async/await patterns; avoid CommonJS artifacts.
- When referencing model relations, follow the naming in models (e.g., `user.hasMany(models.KebeleEmployee, { as: 'kebeleEmployees' })`). Use `db.ModelName` from the dynamic loader, not relative requires.
- Migrations and model enums must be kept in sync (enum values for `role` and `status` are declared in both migration and model). If changing enums, update both migration/seeder and model.

# If you change runtime behavior
- Update `apps/api/migrations/*` (or add new migration) rather than editing past migration files. Prefer new migrations for schema changes.

If anything here is unclear or you want more examples (tests, common controller edits, or a walkthrough for adding a new model + migration), tell me which area to expand and I will update this file.

---
> Source: [FikaduGetachew/cvr](https://github.com/FikaduGetachew/cvr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-28 -->
