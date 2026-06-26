# Copilot Instructions — Carrera Run

## Before completing any change

### 1. Tests

Always check whether tests need to be added or updated:

- **Web** (`apps/web`): Vitest + React Testing Library. Test files live alongside components as `*.test.tsx`.
- **API** (`apps/api`): No test framework is set up yet. When adding non-trivial pure logic (parsing helpers, domain utilities), consider whether a test framework should be introduced.

Rules:
- If you modify existing logic covered by a test, update the test.
- If you add non-trivial logic (parsing, validation, business rules, state transitions), add a test.
- Run existing tests before and after your change: `npm test --workspace @carrera-run/web`.
- **Always review test files after completing a feature** — ask: "Does this change break existing tests? Are there untested code paths?"

### 2. README

Always check `README.md` after every change and update it when:

- A new user-facing feature is added or removed
- An API endpoint is added, changed, or removed
- Environment variables change (also update `.env.example`)
- Setup steps, scripts, or deployment instructions change
- A new import/export format or data contract is introduced

The README is the primary reference for developers and users — keep it accurate.

### 3. Other documentation

- **`.github/agents/`** — Update the relevant agent file when its scope, rules, or referenced knowledge changes.
- **`.github/knowledge/`** — Update coaching methodology or athlete context files when coaching logic or domain rules change.
- **`.env.example`** — Keep in sync with any new or changed environment variables.

### 4. Skills files

After completing a feature or significant change, review whether a new skills file is warranted:

- Skills files live in `.github/skills/` and describe how to use a specific feature or workflow (e.g. how to build a training plan, how to interpret pace zones).
- Create a new skill file when a feature is complex enough that a user or agent would benefit from structured guidance on how to use it.
- Update an existing skill file when the underlying feature changes in a way that makes the guidance stale.
- Ask yourself: "Would an AI agent or a new developer need step-by-step context to use this correctly?"

### 5. Self-updating instructions

After significant changes or repeated patterns observed across sessions, update this file (`copilot-instructions.md`) to capture:

- New conventions adopted (naming, patterns, error handling)
- New tools or libraries added to the stack
- Lessons learned from bugs or regressions (e.g. a class of error to always guard against)
- New rules requested by the team during chat interactions

This keeps Copilot's behaviour consistent across sessions without needing to re-explain context.

## Repository overview

| Area | Path | Notes |
|---|---|---|
| API | `apps/api/src/` | Express + TypeScript, MongoDB |
| Web | `apps/web/src/` | React + TypeScript + Vite |
| Coach prompt | `apps/api/prompts/coach-system.md` | LLM system prompt |
| Knowledge base | `.github/knowledge/` | Coaching methodology & athlete profiles |
| Agent definitions | `.github/agents/` | Running coach and web developer agents |
| Skills | `.github/skills/` | Planning and profile skill guides |

## Git workflow

- **Never push directly to `main`.** Always work on a feature branch and open a pull request.
- Branch names should be descriptive: `feat/`, `fix/`, `chore/` prefixes preferred.
- Keep commits focused and atomic. Write commit messages that explain *why*, not just *what*.
- Every PR should be reviewed before merging — the code reviewer agent is auto-assigned (see `.github/agents/code-reviewer.md`).

## Code style

- Keep route handlers thin — business logic belongs in service modules.
- Never commit secrets or real credentials.
- Use TypeScript types for all API responses and domain models.
- Prefer explicit error handling and user-facing error messages over silent failures.

---
> Source: [zecarrera/carrera-run](https://github.com/zecarrera/carrera-run) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-26 -->
