# GitHub Copilot Instructions

## What Matters Most
- Code centers on the cyclical `useGameStore` simulation in `src/state/useGameStore.ts` that drives resources, nodes, edges, and the main tick loop; always keep state mutations inside the store `set` callback to stay compatible with Zustand middleware.
- Rendering happens through `GraphScene` + `miniplex` (`src/graphics/GraphScene.tsx` and `src/ecs/world.ts`); the store publishes snapshots via `syncGraphWorld`, and the mesh layers read those entities directly, so updates must keep IDs stable if you want R3F transitions to behave.
- UI overlays (`ResourceTray`, `StatsPanel`, `ActionBar`, `SettingsDialog`, `ToastHub`) all pull from `useGameStore`/`useUiStore` selectors; batching selectors together avoids unnecessary re-renders.

## Architecture Reminders
- Global hooks in `App.tsx` (`useGameLoop`, `useAutosave`, `useHydrateFromStorage`) orchestrate side effects. `useGameLoop` runs `step()` every second, `useAutosave` persists through `localStorage` under `AUTOSAVE_KEY`, and `useHydrateFromStorage` restores saves (using `parseGameSave`) before the UI fully renders.
- Serialization helpers live in `src/lib/serialization.ts` (Zod schemas + versioned save payloads). Always use `serializeGameState` before persisting or clipboard exports and `parseGameSave` before hydrating to keep Zod validation intact.
- Numeric helpers (`src/lib/numbers.ts`) and randomness utilities (`src/lib/random.ts`) power resource formatting and procedural decisions—reuse them rather than re-implementing similar logic.
- The `state` folder is TypeScript-first; share types defined in `src/types/graph.ts` when adding new nodes, edges, or stats.

## Workflow Expectations
- Development server: `npm install` → `npm run dev` (Vite with React 19). Expect Fast Refresh when editing `src/` files.
- Builds: run `npm run build` (performs `tsc -b` then Vite build) and `npm run typecheck` to exercise the references model.
- Linting/Formatting: `npm run lint` / `npm run lint:fix` for ESLint rules, and `npm run format` for Prettier; Tailwind utilities live in `tailwind.config.ts`, so verify class names against the declared theme palette.
- Prioritize keeping `@/` path aliases visible when editing to avoid relative path confusion—every TypeScript file imports from `@/` (set in `tsconfig.app.json`).

## Local Patterns & Conventions
- `useGameStore` exposes cost constants such as `EXPLORE_COST`, `NODE_UPGRADE_COST`, and `HYPHAE_UPGRADE_BASE_COST`. Use them in components (`ActionBar`, HUD items) when describing costs or performing checks so tooltips stay accurate.
- Toasts are the common feedback mechanism (`useUiStore.pushToast`), and buttons like `ActionBar` wrap state actions in `runAction` helpers to surface result messages automatically.
- Dialogs (`SettingsDialog`) rely on Radix components; keep the structure compact (overlay + portal + content) and honor the existing styling tokens (rounded 3xl, border-slate, etc.).
- `useUiStore` stores dev toggles such as `showGridOverlay`; the `GraphScene` reads this flag to render a `gridHelper` when debugging ECS alignment, so keep `toggleGridOverlay` accessible in settings.

## Testing & Validation Notes
- Manual validation usually involves spinning up `npm run dev` and performing actions in the UI (Explore, widen hyphae, import/export saves) because the store is the truth source; interactive behavior is hard to unit-test without mocking the Miniplex world.
- When writing automated tests or adding story files, lean on the existing stores and serialization helpers—`serializeGameState` + `parseGameSave` make it easy to capture and restore deterministic graph states.

## Memory Bank & Process
- Follow the Spec-Driven Workflow from `.github/instructions/spec-driven-workflow-v1.instructions.md` alongside the Memory Bank templates (`memory/`). Update `memory/tasks/_index.md` and relevant `TASK###` files whenever you touch requirements, designs, or implementation notes.
- Use `AGENTS.md` + `.github/instructions/memory-bank.instructions.md` for guidance on task ownership, especially if you are filling Build, Research, or Quality roles; document any deviations or decisions in the memory files.

## Final Tips
- Avoid introducing new dependencies unless the value is obvious—this repo already pulls in Radix, Lucide, Zustand, Miniplex, React Three Fiber, and Zod.
- Keep styling within the Tailwind utility system defined in `tailwind.config.ts` and existing CSS layers (noise overlay, base theme), and prefer color names like `soil`, `hyphae`, or `emerald` from the config.
- After modifying state/graph logic, double-check the `GraphScene` output and HUD stats for consistency (tick counter, ResourceTray progress bars, StatsPanel net sugar highlight, toast feedback).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/deadronos) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
