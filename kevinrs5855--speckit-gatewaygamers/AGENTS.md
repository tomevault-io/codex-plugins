# Copilot / AI agent instructions for Speckit

Purpose
-------
Short, targeted instructions to help an automated coding agent (Copilot, CodeBot, etc.) be immediately productive in this repository. This file focuses on the TypeScript/Node backend and React frontend patterns used by Speckit and highlights project-specific conventions an agent must follow.

Quick summary (what to do first)
--------------------------------
- The repository currently has minimal structure. Before making changes, search the workspace for `package.json`, `tsconfig.json`, `src/`, `web/`, `README.md`, and any tests.
- If core project files are missing, create minimal scaffolding aligned with the `speckit.constitution` (TypeScript Node backend + React frontend) and ask maintainers for preferred choices.

Project architecture (expected)
--------------------------------
- Backend: Node.js + TypeScript. Source lives under `server/` or `src/`. Use `express` or a minimal frameworkless approach. Entry point typically `src/index.ts` or `server/index.ts`.
- Frontend: React + TypeScript. Source under `web/`, `frontend/` or `packages/web/`. Use create-react-app, Vite, or Next.js depending on project scale — prefer Vite for minimal free-tier deployment.
- Monorepo: If both backend and frontend exist, prefer a simple two-package layout:
  - `/server` (Node/TS)
  - `/web` (React/TS)
  - Root `package.json` to orchestrate scripts if needed

Key patterns and conventions
----------------------------
- Types and DTOs: Define shared types in `packages/types` or `src/types`. Prefer explicit DTO objects for HTTP endpoints (no implicit any).
- Environment config: Read config from `process.env`. Provide `config.example` or `.env.example` at the root.
- HTTP APIs: Use resource-based routes (`/api/items`, `/api/items/:id`). Return JSON `{ data: ..., error?: { code, message } }` for consistency.
- Error handling: Use centralized error middleware in Express (`src/middleware/error.ts`) that maps internal errors to HTTP status codes and logs structured JSON.
- Testing: Backend tests in `tests/` using `jest` + `ts-jest` or `vitest`. Frontend tests use `jest` + `react-testing-library` or `vitest` with `@testing-library/react`.

Build, test, and run commands (examples)
----------------------------------------
If there is no `package.json`, create one like the examples below. Use these commands as canonical unless repository files say otherwise.

Root package.json scripts (example for a two-package layout):

- Install dependencies:

```bash
npm install
```

- Backend dev:

```bash
cd server
npm run dev # typically `ts-node-dev --respawn src/index.ts` or `vite-node` for Vite-based setups
```

- Frontend dev:

```bash
cd web
npm run dev # typically `vite` or `react-scripts start`
```

- Run tests (root runs both):

```bash
npm test
```

- Lint/format:

```bash
npm run lint
npm run format
```

Project-specific code guidance (do this, not that)
--------------------------------------------------
- Do: Export small, well-typed functions from modules. Add JSDoc/TSDoc comments for public functions.
- Do: Create `src/controllers`, `src/services`, `src/repositories` for separation of concerns.
- Do: Use `async/await` and avoid mixing callbacks and promises.
- Don't: Add large, un-reviewed dependencies. Prefer lightweight libraries compatible with free-tier deployments.

Examples drawn from repository (if present)
-------------------------------------------
- If you add a new API route, mirror it in `web/src/api/` with a typed client wrapper that uses `fetch`.
- If you create a database migration, place it in `server/migrations/` and document how to run it in `README.md`.

Observability and CI
--------------------
- Add structured logging (JSON). Include `requestId` headers for cross-service tracing.
- CI should run `npm run lint` and `npm test`. Integration tests that call cloud services must be gated behind an opt-in workflow and documented.

When to ask the maintainers
---------------------------
- If the repository is empty and you intend to scaffold the project, ask: preferred framework (Express/Koa/Nest/Vite/Next), CI provider, and coding style (Prettier/ESLint config).
- If a design decision will increase runtime cost (e.g., adding a DB), ask for approval and an ADR.

Merging and PR rules for agents
-------------------------------
- Create focused PRs with a clear title and description. Include a short checklist linking to items in `speckit.constitution`.
- Run unit tests and linters locally before creating the PR.
- Assign a human reviewer; do not merge without at least one human approval.

How to extend these instructions
-------------------------------
- If you add a framework or major dependency, update this file with the canonical commands and examples.
- Keep examples minimal and specific to the code discovered in the repo.

Contact & escalation
--------------------
If you need a decision, open a short issue or draft PR and request guidance from the maintainers. For time-sensitive infra or security changes, ping the on-call maintainer.

---

Notes
-----
- These instructions prefer TypeScript/Node and React patterns per maintainer request. If you find existing code using a different stack, adjust to that stack and notify maintainers.
- Keep outputs and changes minimal and well-documented.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kevinrs5855)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/kevinrs5855)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
