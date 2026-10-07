# Copilot Instructions for Work-Timer

## Repository Workflow

- Treat `master` as production-ready at all times.
- Never perform feature development directly on `master`.
- Create a short-lived branch from `master` for each task.
- Open a PR to merge back into `master`.
- Merge only after checks pass (`npm test` and `npm run build`).
- Prefer squash merge.

## Branch Naming

- `feature/<short-description>`
- `fix/<short-description>`
- `chore/<short-description>`

## Development Guardrails

- Keep changes focused and minimal.
- Do not bypass safety behavior in timer, billing, export, or destructive flows.
- Preserve current CLI and MCP behavior unless the task explicitly changes it.
- For user-facing PowerShell/terminal output formatting, follow `docs/powershell-output-style-guide.md` as the default standard for current and future output UX changes.

## Safety And Security First Policy

- Treat user data protection, billing correctness, and destructive-operation safety as top priority over speed.
- Default to least-privilege and non-destructive behavior: prefer dry-run, explicit confirmations, and reversible changes.
- Never weaken or bypass safety checks, authorization boundaries, or validation logic unless explicitly required and documented.
- If a change introduces meaningful security, privacy, or integrity risk, stop and escalate with options, risks, and a recommended safe default.
- Call out security-impacting changes in PR summaries and include focused validation steps.

## Local Staging Workspace Rules

- Use `staging/` for local-only test notes, debug helpers, and temporary artifacts.
- Treat `staging/` as non-production and non-release content.
- Do not commit files from `staging/`.
- Do not reference `staging/` as a required documentation location; durable docs must live under `docs/`.

## Required Validation Before PR

- Run `npm test` and `npm run build`.
- Run `npm run check:production` to catch debug/dev-only artifacts before merge.
- If source files change under `src/`, ensure generated outputs under `dist/` are regenerated and committed when applicable.
- If CLI command behavior changes, update docs in `docs/cli-reference.md` and relevant setup/readme pages.
- If MCP tools change, update `docs/mcp-tools.md` to keep tool docs and counts accurate.

## MCP Installer And Client Integration Rules

- Keep installer behavior safe by default: support dry-run, avoid destructive overwrites, and keep backup behavior for existing JSON configs.
- When adding/removing supported MCP clients, update all of:
	- installer logic under `src/cli/mcp-install.ts`
	- user docs (`docs/setup.md`, `docs/cli-reference.md`, and `README.md` as needed)
	- client watchlist manifest under `.github/llm-client-watchlist/package.json` when relevant

## Release Hygiene

- Keep version references in sync when cutting a release change:
	- `package.json`
	- `package-lock.json`
	- `src/cli/index.ts` (CLI version)
	- `src/mcp/server.ts` (MCP server version)
- Add a clear entry in `CHANGELOG.md` for user-visible changes.

## Merge Gate

- Version tags are cut from `master` only.
- If a PR is not production-ready, do not merge it.

## Merge Administration Policy

- The coding agent administers merge readiness by default.
- The agent should run review + validation, decide when the outcome is clear, and proceed without waiting for user confirmation in routine cases.
- The agent should escalate to the user only when there is a material choice (trade-off between valid options), unresolved risk, or policy ambiguity.
- When escalating, present concise options, risks, and a recommended default.

## PR Review Automation

- When a PR is in a `REVIEW_REQUIRED` state, the active coding agent must run a review automatically.
- The review must prioritize risks/regressions first (correctness, safety, behavior changes, and missing tests).
- The agent must provide clear merge recommendations (approve/changes requested) with concrete follow-up actions.

---
> Source: [JoelBondoux/Work-Timer](https://github.com/JoelBondoux/Work-Timer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-10-07 -->
