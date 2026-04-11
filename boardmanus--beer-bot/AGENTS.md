# Copilot Onboarding Instructions for beer-bot

## Repo overview
- Purpose: beer-bot is a small IoT/beer-tracker server + client system for interacting with Tilt hydrometers (Bluetooth devices) and serving a web UI.
- Stack: Node.js (TypeScript), Express-like server rendering Pug views, simple REST/WebSocket interaction from `src/server`, and client web assets in `src/public`.
- Main code paths:
  - `src/server/server.ts`: app entry and API.
  - `src/server/config.ts`: config load and validation.
  - `src/common/tiltpayload.ts`, `src/common/utils.ts`: shared parsing/logic.
  - `src/client/client.ts`: browser-side control.

## Assistant style

- Output style: neutral, factual, informative with no extra commentary
- Always answer with direct action items only
- When addressing problems, justify reasoning

## Project type and size
- Single repo (monorepo-style with `src/client`, `src/common`, `src/server`).
- Lightweight codebase (~300 LOC core, legacy dist for built artifacts).
- TypeScript + tests with Jest (or ts-jest via `pnpm test`).

## Setup and toolchain
- Required runtime: Node.js 18+ (LTS). Verify with `node -v`.
- Package manager: `pnpm` (ideally v8+). Verify with `pnpm -v`.

## Bootstrap / install
- Always run:
  - `pnpm install`
- If switching branches or after `git pull`: repeat install.

## Build
- Build command (validated): `pnpm build`.
- Dev build: `pnpm build:dev`.
- Outputs written to `dist-dev/` and `dist-pro/`.

## Test
- Run unit tests with: `pnpm test`.
- For specific files: `pnpm test -- src/server/test/config.test.ts`.

## lint / quality checks
- No explicit lint script in package.json; rely on `pnpm check` and TypeScript compile.
- Validate TypeScript compile via `pnpm check`.

## Run
- Launch server: `pnpm run:dev` after build.
- In development, run: `pnpm build:dev && pnpm run:dev`.

## Common commands and safe ops
- `git status` before staging/commits.
- `pnpm install` before any build/test.

## Pre-commit/CI best practices
- Check `package.json` scripts and run all relevant commands (build/test).
- Keep modifications in `src/`; do not edit `dist-*` generated output.

## Architecture notes for code changes
- backend: `src/server/**`, frontend: `src/client/**`, shared: `src/common/**`.
- config: `src/server/config.ts`.
- tests: `src/server/test`, `src/common/test`.

## Documentation and discovery
- Use `README.md` for high-level usage instructions.
- Search TODO/HACK comments via `grep -R "TODO\|HACK" src`.

## Trust policy for agent runs
- Trust this guidance first; only use code search tools if the instruction is outdated or a new script is observed.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/boardmanus)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/boardmanus)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
