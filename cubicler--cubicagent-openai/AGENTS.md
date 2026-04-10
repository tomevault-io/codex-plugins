# Repository Guidelines

## Project Structure & Module Organization

- Source: `src/` (core logic in `src/core/`, config in `src/config/`, internal tools in `src/internal-tools/`, helpers in `src/utils/`, types in `src/models/`).
- Entry point: `src/index.ts` → compiled to `dist/index.js`.
- Tests: `tests/unit/` and `tests/integration/` with setup in `tests/setup.ts`.
- Build output: `dist/` (ignored by lint/tests).

## Build, Test, and Development Commands

- `npm run dev`: Start TypeScript in watch mode via `tsx`.
- `npm run build`: Type-check and compile to `dist/` using `tsc`.
- `npm start`: Run compiled app (`dist/index.js`). Also the CLI bin `cubicagent-openai` points here.
- `npm test`: Run all tests with Vitest. Variants: `test:unit`, `test:integration`, `test:watch`.
- `npm run test:coverage`: Generate coverage (text, lcov, html).
- `npm run lint` / `lint:fix`: Check/fix lint issues in `src/**/*.ts`.
- Docker: see `DOCKER.md` and `docker-compose.yml` for containerized runs.

## Coding Style & Naming Conventions

- Language: TypeScript (ES2022, strict mode on). Path alias: `@/*` → `src/*`.
- Linting: ESLint with `@typescript-eslint` (see `eslint.config.js`). No unused vars, prefer `const`, no `var`.
- Files: kebab-case for filenames (e.g., `openai-service.ts`), `.interface.ts` for interfaces.
- Indentation & formatting: follow ESLint autofix; keep imports ordered logically.
- Public APIs: export explicit types from `src/models` and `src/config` where relevant.

## Testing Guidelines

- Framework: Vitest (`vitest.config.ts`). Test files: `tests/**/*.{test,spec}.ts`.
- Structure: mirror source folders under `tests/unit` and `tests/integration`.
- Conventions: name tests after target module (e.g., `openai-service.test.ts`). Keep deterministic, use `tests/setup.ts` for globals.
- Coverage: run `npm run test:coverage`; include `src/**/*.ts` only.

## Commit & Pull Request Guidelines

- Commits: use Conventional Commits when possible (`feat:`, `fix:`, `chore:`, `docs:`, `refactor:`, `test:`, `ci:`). Keep messages imperative and scoped.
- PRs: include clear description, rationale, and screenshots/log excerpts if behavior changes. Link issues, list notable env/config changes, and update `README.md`/`.env.example` as needed.
- Quality gate: PRs should pass lint, tests, and include tests for new logic.

## Security & Configuration Tips

- Use `.env` (see `.env.example`). Do not commit secrets. Required: `OPENAI_API_KEY`; set transport via `TRANSPORT_MODE` (`http`|`stdio`|`sse`) and related vars (e.g., `CUBICLER_URL`, `SSE_URL`).
- Validate config with `loadConfig()` (`src/config/environment.ts`); prefer updating schemas when adding vars.

## Engineering Principles

- Architecture: contract-based, SOLID, and dependency inversion; avoid clever hacks.
- Code: TypeScript-first with explicit types; prefer small, pure functions; justify any mutation.
- Errors: never swallow; bubble via typed errors or a Result/Either style.
- Tests: TDD-leaning; propose and add unit tests first (Vitest by default).
- Docs: write short JSDoc for public functions; add an ADR note when changing architecture.
- Security: validate inputs and sanitize external data; never commit secrets.
- Git: make atomic commits with clear messages; show planned diff before writing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Cubicler)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Cubicler)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
