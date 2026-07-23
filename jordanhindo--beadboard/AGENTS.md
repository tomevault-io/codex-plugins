# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What Is BeadBoard

BeadBoard is a Windows-native local dashboard for the [Beads](https://github.com/steveyegge/beads) issue tracker. It reads `.beads/issues.jsonl` files directly from repos and renders them as an **agent-first operations console**: a unified shell with Social, Graph, and Activity lenses for monitoring multi-agent work, managing swarms, and intervening on blockers. It supports multi-project aggregation, real-time file watching with SSE, and swarm/archetype coordination.

**Active UX implementation spec**: `docs/plans/2026-02-28-ux-redesign-synthesis-prd.md`

## Commands

```bash
npm run dev          # Start Next.js dev server (localhost:3000)
npm run build        # Production build
npm run typecheck    # tsc --noEmit
npm run lint         # eslint
npm run test         # Runs all tests (explicitly enumerated in package.json)
```

Run a single test:
```bash
node --import tsx --test tests/lib/parser.test.ts
```

**New test files must be added to the `test` script in `package.json`** — the suite is explicitly enumerated, not auto-discovered.

## Verification gates (run before closing any code bead)

```bash
npm run typecheck && npm run lint && npm run test
```

## Architecture

**Stack**: Next.js 15 (App Router) · React 19 · Tailwind CSS · TypeScript (strict) · @xyflow/react (graphs) · Dagre (layout) · Chokidar (file watching) · Remotion (video generation)

### Data flow

1. **Source of truth**: `.beads/issues.jsonl` files on disk, mutated only via `bd` CLI commands.
2. **Parser** (`src/lib/parser.ts`): Reads JSONL, normalizes dependencies and fields into `BeadIssue` objects.
3. **Aggregate read** (`src/lib/aggregate-read.ts`): Merges issues across multiple project roots based on project scope/mode.
4. **Realtime** (`src/lib/realtime.ts`): `IssuesEventBus` singleton dispatches change events. SSE endpoint (`src/app/api/events/route.ts`) streams them to the browser.
5. **Watcher** (`src/lib/watcher.ts`): Chokidar-based `IssuesWatchManager` watches JSONL + WAL + `.last_touched` files, coalesces events via `ProjectEventCoalescer`, and emits through the event bus. Snapshot diffing detects per-issue changes.
6. **Writeback** (`src/lib/writeback.ts`): API mutations write back to JSONL through `bd`-compatible paths.

### Key directories

- `src/app/` — Next.js App Router pages and API routes
  - `/` — **Primary entry point.** `UnifiedShell` with query-param-driven views: `?view=social|graph|activity`
  - `/graph`, `/sessions`, `/timeline` — Legacy routes. Still exist but are not linked from the unified shell. Do not add new features here; migrate functionality into the unified shell instead.
  - `/api/beads/` — CRUD endpoints for beads (create, read, update, close, reopen, comment)
  - `/api/events/` — SSE endpoint for real-time updates
  - `/api/swarm/` — Swarm coordination endpoints (create, launch, join, leave, status, templates)
  - `/api/mission/` — Mission/epic management endpoints
  - `/api/agents/` — Agent registry endpoints
- `src/components/` — UI components organized by feature (kanban, graph, sessions, swarm, shared, activity, etc.)
  - `shared/unified-shell.tsx` — **Root layout.** Top bar, left panel, middle content (view-switched), right panel, thread drawer
  - `shared/left-panel.tsx` — Navigation spine: view switcher (Social/Graph/Activity), epic tree, filters
  - `shared/top-bar.tsx` — Global header: identity, health, blocked triage, metric tiles
  - `activity/contextual-right-panel.tsx` — Right panel content router (branches on URL context)
  - `activity/activity-panel.tsx` — Activity/telemetry feed (also used as the `activity` view content)
  - `graph/smart-dag.tsx` — DAG renderer using @xyflow/react + Dagre
  - `social/social-page.tsx` — Social lens: grouped task cards with dependency/thread context
- `src/lib/` — Core logic (parser, types, pathing, watcher, realtime, registry, scanner, mutations, graph layout)
  - `types.ts` — Central type definitions (`BeadIssue`, `BeadStatus`, `ProjectContext`, etc.)
  - `pathing.ts` — Windows path canonicalization (drive letter normalization, path keys)
  - `bd-path.ts` — Resolves the `bd` CLI binary path
- `src/hooks/` — React hooks for subscriptions (`use-beads-subscription`, `use-swarm-list`, `use-url-state`, etc.)
- `components/ui/` — shadcn/ui primitives
- `tests/` — Node.js native test runner with tsx loader; mirrors src structure
- `scripts/` — CLI utilities (`bb.ts` — BeadBoard CLI, Playwright capture scripts)
- `docs/` — RFCs, ADRs, protocols, and feature specs

### Path alias

`@/*` maps to the project root (configured in `tsconfig.json`).

### Windows considerations

- All path handling must canonicalize Windows paths (drive letter casing, backslash normalization). Use `canonicalizeWindowsPath` and `windowsPathKey` from `src/lib/pathing.ts`.
- Webpack filesystem cache is disabled in dev mode (`next.config.ts`) to avoid Windows ENOENT errors.

### ESLint config

`@typescript-eslint/no-explicit-any` and `prefer-const` are intentionally turned off. `eslint.config.mjs` uses flat config with `next/core-web-vitals` and `next/typescript`.

## Beads workflow

Use `bd` as the sole interface for mutating beads. Never write to `.beads/issues.jsonl` directly. See `AGENTS.md` for the full operating manual including dependency discipline, parallel agent patterns, and session completion protocol.

---
> Source: [jordanhindo/beadboard](https://github.com/jordanhindo/beadboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-23 -->
