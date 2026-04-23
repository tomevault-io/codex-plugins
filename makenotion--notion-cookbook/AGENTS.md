<!-- .github/copilot-instructions.md - guidance for AI coding agents -->

# Copilot instructions for contributors and AI agents

Purpose

- Give concise, actionable guidance so an AI assistant can be productive in this repo.

Quick context

- This repo is a collection of Notion examples and AI "skills" (Claude/MCP). See the top-level README for intent.
- Important areas:
  - examples/javascript/ — runnable TypeScript/Node examples (each example has its own README and often an example.env).
  - skills/claude/ — AI skill definitions. Each skill contains a `SKILL.md` (authoring) and an `evaluations/` folder (test scenarios).
  - docs/ — additional developer docs and MCP integration notes.
  - scripts/ — helper scripts: `install-examples.sh` and `typecheck-examples.sh`.

What to do first

- Read `skills/README.md` and `skills/claude/README.md` to understand conventions for SKILL.md and evaluations.
- Open the example's own README before making changes to an example (examples are self-contained).

Project-specific workflows

- Examples: change into the example folder, run `npm install`, then follow the example README. Many examples expose a `ts-run` script (e.g. examples/javascript/generate-random-data/package.json uses `npm run ts-run`). Example env files are named `example.env` — copy to `.env` and fill secrets.
- Type checks: use `scripts/typecheck-examples.sh` to run TypeScript checks across examples.
- Bulk setup: `scripts/install-examples.sh` installs dependencies for examples; prefer it when adding or updating many examples.

Conventions and patterns

- Examples are lightweight, TypeScript-first, often using `ts-node` for quick runs. Respect the example's package.json scripts rather than adding global runners.
- Skills: `SKILL.md` is authoritative for the AI flow. Evaluations in `evaluations/` are small JSON scenarios used to validate behavior — modify or add evaluations when changing a skill.
- Keep SKILL.md concise (<500 lines) and place detailed references in separate files (as the repo already does in `skills/*/reference`).

Integration points and external dependencies

- Notion API: examples rely on `@notionhq/client` (check the examples' package.json).
- MCP/Claude: skills assume a configured Notion MCP server and Claude environment. See `skills/claude/README.md` for install/usage notes.

Editing guidance for AI agents

- When editing examples, run the example's README steps and the local `ts-run` (or `npm start`) to verify behavior where feasible.
- For skills, update or add evaluations alongside any SKILL.md change. Evaluations are the minimal runnable spec for verifying the behavior.
- Prefer small, focused commits that update a single example or skill and its tests/evaluations.

Files to inspect when beginning work

- [scripts/install-examples.sh](scripts/install-examples.sh)
- [scripts/typecheck-examples.sh](scripts/typecheck-examples.sh)
- examples/javascript/\*/README.md (per-example instructions)
- skills/claude/_/SKILL.md and skills/claude/_/evaluations/

If something's unclear

- Ask: which example or skill should be the focus and whether you should run the example locally.

Please review and tell me any sections that need more detail or examples.

---
> Source: [makenotion/notion-cookbook](https://github.com/makenotion/notion-cookbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-22 -->
