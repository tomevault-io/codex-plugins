# Copilot Instructions — keyoku

## Project Overview

TypeScript monorepo (npm workspaces) providing persistent AI memory for OpenClaw agents. Three packages:

- **@keyoku/types** — Shared type definitions (Memory, SearchResult, HeartbeatResult, etc.)
- **@keyoku/memory** — HTTP client library for keyoku-engine API (zero runtime dependencies)
- **@keyoku/openclaw** — OpenClaw plugin (hooks, tools, init wizard, CLI, migrations)

## Architecture

- Plugin registers OpenClaw lifecycle hooks: `before_prompt_build` (auto-recall + heartbeat injection) and `agent_end` (incremental capture of user+assistant pairs)
- 7 registered tools: memory_search, memory_get, memory_store, memory_forget, memory_stats, schedule_create, schedule_list
- Entity resolution supports 5 strategies: static, per-user, per-channel, per-session, template
- Keyoku engine runs as a Go sidecar subprocess managed by `service.ts`
- Communication is HTTP REST on port 18900 with Bearer token auth

## Code Style

- TypeScript strict mode everywhere
- Prettier: single quotes, trailing commas, 100 char width, 2-space indent
- ESLint: recommended + typescript-eslint strict
- Conventional commits: feat/, fix/, docs/, test/, chore/

## Package Rules

- @keyoku/types and @keyoku/memory must have zero runtime dependencies
- @keyoku/openclaw peer-depends on openclaw (optional)
- All three packages share the same version number — bump together
- Every package builds with `tsc` and emits .d.ts declarations + source maps

## Testing

- Framework: vitest (globals enabled, 10s timeout)
- Tests live in `packages/*/test/*.test.ts`
- Mock network calls with `vi.fn()` and `vi.stubGlobal()` — never make real HTTP calls in tests
- New features must have tests

## Plugin Safety

- `capture.ts` contains prompt injection detection patterns — do not weaken these
- Always use `resolver.isAllowed()` before any memory operation in group contexts
- Always resolve entityId via `resolver.resolve()` — never hardcode entity IDs in tools
- Respect the 3 autonomy levels (observe/suggest/act) — never bypass user autonomy settings
- `api.logger.debug?.()` for startup messages, never `info` — info pollutes ACP bridge stdout (JSONL protocol)

## Init Wizard (init.ts)

- Multi-step interactive setup: config detection, binary install, plugin registration, LLM provider selection, engine compatibility check, autonomy, timezone, heartbeat, skill install, migration, health check
- Supports migration from OpenClaw MEMORY.md, LanceDB vector stores, and HEARTBEAT.md cron rules
- Engine compatibility gate: checks installed binary version against provider requirements before proceeding
- ANSI color output via the `c` helper object — keep formatted output consistent

## CI/CD

- `ci.yml`: runs on push/PR to main — install, build, test (Node 20)
- `e2e.yml`: manual dispatch — Docker Compose integration tests with real API keys
- `release.yml`: on v* tags — build, test, publish all 3 packages to npm, create GitHub release
- All tests must pass before merge

## What to Flag

- Any change to hook priorities (before_prompt_build is -10)
- Changes to entity resolution logic or strategy defaults
- New info-level logs in plugin registration path (breaks ACP)
- Removal or weakening of prompt injection patterns in capture.ts
- Direct fetch calls in @keyoku/memory that don't go through the client's error handling
- Missing entityId resolution in tool handlers
- Version number mismatches across packages

---
> Source: [Keyoku-ai/keyoku](https://github.com/Keyoku-ai/keyoku) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-03 -->
