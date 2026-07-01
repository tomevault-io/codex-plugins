<!--
Project-specific Copilot instructions.
Keep this short (20-50 lines) and focused on discoverable patterns, workflows and files that help AI agents be productive.
-->

# Quick orientation

- This workspace is a small scaffold used to host project files and agent guidance. Root files:
	- `src/` — source code (entry: `src/index.js`)
	- `tests/` — test runner script (`tests/sample.test.js`)
	- `package.json` — basic scripts (`npm test` runs the simple test)

# What to look for (big-picture)

- There is no complex architecture yet. For larger projects, look for top-level folders like `api/`, `web/`, `services/`, `infra/`, or `packages/`.
- When present, trace the runtime entrypoints (`server.js`, `src/index.js`, `bin/`) and read corresponding README or `package.json` scripts to understand launches.

# Developer workflows (commands you can run)

- Install / validate (if node project): `npm install` then `npm test` — when present, prefer the project's `scripts` in `package.json`.
- VS Code tasks are configured in `.vscode/tasks.json` when added; otherwise use npm scripts or direct node/python commands.

# Project-specific conventions

- Formatting: this workspace includes `.vscode/settings.json` with `editor.formatOnSave` enabled; follow project's formatter if present (Prettier/ESLint configs).
- Tests: the repo uses a lightweight test command in `package.json` that runs `tests/sample.test.js`. For real projects, look for `jest`, `mocha`, or `pytest` configs.

# Integration & external tools

- External services (databases, APIs, cloud) are documented in each service's README. Search for `.env`, `docker-compose.yml`, or cloud config files to locate integrations.

# How AI agents should modify code here

- Make minimal, well-scoped edits. When adding files, update `README.md` and `package.json` scripts if they change developer workflows.
- Prefer to also add or update a test under `tests/` for any behaviour change and run `npm test`.

# Enabling Claude Sonnet 3.5 / agent runtime notes

- Enabling or configuring Claude Sonnet 3.5 for clients is an administrative action outside this repo (requires platform-level access or VS Code extension settings). Document steps here:
	1. Confirm tenant or org-level access to enable model for clients (not done via repo files).
	2. If using a VS Code extension that exposes model selection, add step-by-step instructions to `AGENT.md` and note any required extension IDs.
 3. If the workspace must programmatically call Claude APIs, place credentials in a secrets manager and document env var names (do NOT store secrets in repo).

# Local environment / home directory note

- User requested home directory: set workspace root to `/Users/agafoor/Development/VSCodeHome`. Agents should use relative paths from repo root; avoid hardcoding absolute paths.

# References (files to open first)

- `README.md` — project summary
- `.github/copilot-instructions.md` — this file
- `AGENT.md` — supplementary agent notes
- `package.json`, `src/index.js`, `tests/sample.test.js` — quick runtime and test examples

# If anything is unclear

- Add a short question as an issue or leave a comment in `AGENT.md`. Ask for missing architecture details, env variable names, or CI credentials.

---
> Source: [agafoor/vscodehome](https://github.com/agafoor/vscodehome) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-01 -->
