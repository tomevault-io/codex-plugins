# Copilot Instructions — Just Roll Crits

## Mission
This repository is a Star Wars: Legion attack/damage calculator with both deterministic and Monte Carlo analysis.

When generating or editing code:
- Prioritize **rules correctness** over UI polish.
- Keep logic changes **small, testable, and explicit**.
- Preserve existing architecture boundaries (engine ↔ state ↔ UI).

## Tech Stack & Runtime
- **Frontend:** React 18 + TypeScript (strict mode)
- **Build tool:** Vite
- **Styling:** Tailwind CSS (dark UI design language)
- **State management:** Zustand
- **Testing:** Vitest + React Testing Library
- **Data viz:** Recharts
- **PWA:** `vite-plugin-pwa`
- **Desktop packaging:** Tauri (`src-tauri/`)

## Repository Map
- `src/` — web app entry, layout, app composition
- `src/components/` — UI panels and shared components
	- `AttackerPanel/`, `DefenderPanel/`, `ResultsPanel/`, `AttackTypeSelector/`, `shared/`
- `src/engine/` — pure game logic (dice math, sequence steps, modifiers, simulation)
	- `worker/` — Web Worker for off-thread Monte Carlo simulation
- `src/stores/` — Zustand stores and selectors
- `src/hooks/` — React hooks (simulation lifecycle, display helpers, keyword restrictions)
- `src/data/` — 5-stage data pipeline (fetch → process → enrich → resolve → presets)
	- `raw/` — API snapshot JSON (8 files from TableTopAdmiral)
	- `processed/` — cleaned/slugified data (units, upgrades, keywords)
	- `enrichment/` — manually curated weapon profiles, surge charts, keyword values
	- `upgradeApplicator.ts` — bridge between data layer and engine configs
- `src/utils/` — generic helpers (formatting, keyword restrictions, weapon counts)
- `src/test/` — shared test setup/utilities
- `src/integration/` — full-pipeline integration tests
- `scripts/` — data pipeline scripts and validation tooling (run via `npx tsx`)
- `rulebook_markdown/` — source material for rule interpretation
- `plans/` — implementation plans/history and design intent
- `src-tauri/` — desktop wrapper and Rust/Tauri config

## Architectural Rules

### Engine (`src/engine`) is pure
- No React imports.
- No direct Zustand/store access.
- No DOM, browser APIs, random global state, or side effects.
- Prefer deterministic functions with explicit inputs/outputs.
- Engine functions never `throw` — use `Math.max(0, ...)` clamping to prevent negative values.
- Randomness is localized to `dice.ts` only. All other engine functions are deterministic given their inputs.

### Data layer (`src/data`) is the adapter
- Transforms external API data into engine-compatible configs.
- No React imports, no direct store access.
- May import from `src/engine/types.ts` for type definitions.
- The `upgradeApplicator.ts` module is the critical bridge between data and engine.
- Resolvers and preset generators use lazy-initialized cached singletons (no cache invalidation — app reload required for data changes).

### UI and Store responsibilities
- Components handle presentation and user interaction.
- Zustand store manages app/session state.
- Derived combat results should be computed by engine functions, not duplicated in components.
- Components should not call engine functions directly — use hooks or selectors as the bridge.

### Data flow
- User input/state → normalize/validate → engine calculation/simulation → presentation.
- The selector pipeline: `selectConfig → applyUpgrades → rebuildWeapons → applyOverrides → engine`.
- Keep conversion/normalization close to boundaries (store or adapter helpers), not scattered across components.

## Domain Modeling Expectations
- Model the Legion attack sequence in clear stages (attack roll, surges, modifiers, cover/dodge, defense roll, pierce/blocks, wounds).
- Keep keyword interactions explicit (Impact, Pierce, Critical, Armor, Cover, Dodge, surge conversions, etc.).
- Avoid “magic” transformations; name intermediate values so combat steps are auditable.
- If a rule interaction is ambiguous, prefer the existing engine/test behavior and note assumptions in tests.
- Use `rulebook_markdown/` as the authoritative in-repo rules reference.

## TypeScript & API Design Guidelines
- Keep TypeScript strict-safe; prefer avoiding `any` (note: ESLint does not currently enforce this — `@typescript-eslint/no-explicit-any` is off).
- **Always use enum members** (e.g., `DefenseDieColor.White`, `CoverType.None`) instead of raw string or numeric literals when a type is backed by an enum. Never use a bare `'white'`, `'none'`, `0`, etc. where the type expects an enum value.
- Use discriminated unions or explicit types for mode-specific behavior.
- Prefer small pure helper functions over large monolithic procedures.
- Keep public function signatures stable unless a broader refactor is required.
- Use named exports for components/utilities; default exports only for page-level shells (`App`, `Layout`).

## React & UI Conventions
- Functional components only.
- Co-locate component tests with component code (`*.test.tsx`).
- Use Tailwind utilities for styling; avoid CSS modules/styled-components.
- Reuse shared UI components before introducing new primitives.
- Keep the dark-theme design consistent; do not introduce ad-hoc color systems.

## Testing Requirements
- Co-locate tests as `*.test.ts` / `*.test.tsx`.
- Engine changes should include or update focused unit tests in `src/engine/`.
- UI behavior changes should include React Testing Library coverage where practical.
- Prefer targeted tests first, then broader suites.
- Do not rewrite large snapshots unnecessarily.

## Quality Bar for Changes
- Fix root causes, not surface symptoms.
- Keep diffs minimal and scoped to the requested behavior.
- Avoid unrelated refactors while implementing feature/fix work.
- Maintain naming and file organization patterns already used in nearby code.
- Update docs when behavior or workflow meaningfully changes.

## Performance & Simulation Notes
- Keep Monte Carlo logic efficient and side-effect free.
- Avoid unnecessary allocations in hot loops.
- Keep deterministic and simulation pathways behaviorally aligned where intended.
- If adding random behavior, ensure seeding/control patterns remain testable.
- Simulation runs in a Web Worker (`src/engine/worker/`). The worker client uses request-ID tracking to discard stale results.
- The deterministic EV estimator (`woundEstimation.ts`) is used by the Marksman decision engine, not by the main simulation path.

## Error Handling Policy
- Engine functions **never throw** — clamp values with `Math.max(0, ...)` instead of throwing on invalid inputs.
- Simulation errors flow through a structured pipeline: Worker `try/catch` → Worker client `reject` → `useSimulation` hook `catch` → `resultsStore.setError()` → `ErrorDisplay` component.
- Store actions are no-ops when guards fail (e.g., `appendResult` is a no-op when slots are full).
- There is currently **no React Error Boundary** — an unhandled component exception will crash the app.
- Console logging is minimal (only 1 `console.warn` in the entire `src/` tree). Do not add `console.log` statements.

## Build, Test, and Dev Commands
Use these standard workflows unless task-specific instructions say otherwise:
- `npm run dev` — local web development (port 8080)
- `npm run lint` — REQUIRED for all code changes (must pass with 0 errors)
- `npm run typecheck` — REQUIRED for all code changes (must pass with 0 errors)
- `npm run test` — run test suite (watch mode)
- `npm run test:run` — single-pass test execution
- `npm run test:coverage` — test with coverage report
- `npm run test:e2e` — Playwright end-to-end tests
- `npm run build` — production build
- `npm run preview` — preview built web app
- `npx tsx scripts/<name>.ts` — run data pipeline or validation scripts
- Tauri commands should follow existing docs (`TAURI_QUICKSTART.md`, `DESKTOP_DEPLOYMENT.md`).

## Guidance for Copilot-generated Changes
When asked to implement work:
1. Identify whether the change belongs in engine, store, UI, or data.
2. Implement with minimal API surface changes.
3. Add/update tests nearest to changed logic.
4. Run `npm run typecheck` and `npm run lint`.
5. Run relevant tests.
6. Summarize assumptions, especially for rules interactions.

### Mandatory Quality Gate
- Do not treat a change as complete if `npm run typecheck` or `npm run lint` fails.
- Fix introduced TypeScript/lint issues before finalizing.
- If unrelated pre-existing failures block completion, explicitly report them and scope them apart from your change.

For bug fixes:
- Add or adjust a failing test that demonstrates the bug when feasible.
- Then implement the fix.

For feature work:
- Prefer extending existing modules over creating new abstractions unless duplication is clear.

## Source of Truth Priority
When instructions conflict, use this precedence:
1. Direct user request
2. This `copilot-instructions.md`
3. Existing code patterns/tests
4. Plan/docs in `plans/` and top-level markdown docs

## Non-Goals
- Do not add new frameworks.
- Do not migrate state management away from Zustand.
- Do not introduce class components.
- Do not move engine logic into UI components.

## Scoped Instruction Files

For deeper guidance on specific subsystems, see the scoped instructions in `.github/instructions/`:

| File | Scope | Covers |
|------|-------|--------|
| `engine.instructions.md` | `src/engine/**` | Attack sequence steps, dual analysis modes, test helpers, purity constraints |
| `data-pipeline.instructions.md` | `src/data/**`, `scripts/**` | 5-stage pipeline, enrichment workflow, resolver caching, script inventory |
| `stores.instructions.md` | `src/stores/**`, `src/hooks/**` | 4-store architecture, selector pipeline, stale tracking, dead stub files |
| `components.instructions.md` | `src/components/**` | Panel architecture, shared component inventory, dark theme palette, accessibility |
| `keywords.instructions.md` | Cross-cutting | End-to-end keyword addition checklist (7+ files), aggregation rules, restrictions |
| `testing.instructions.md` | All test files | Engine/component/store/E2E patterns, factory usage, coverage expectations |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ben-lear)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/ben-lear)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
