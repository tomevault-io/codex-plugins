# LeanLog Copilot Instructions

## Product

- LeanLog is a calm, local-first weight-loss tracker for real life.
- The current app ships five primary surfaces: Dashboard, History, Averages, About, and Settings.
- Daily logs track weight, calories, steps, exercise type, and exercise minutes.
- Tracking defaults also include a one-time height setting used only for derived BMI guidance in Health Journey; height is never logged daily.
- The dashboard stays focused on five sections: Today's Snapshot, Weight Trend, Daily Consistency, Progress Toward Your Goal, and Health Journey.
- Mobile drawer flows are touch-first and should stay compact, keyboard-light, and calm.
- Keep the product psychologically light, forgiving of missed days, and free of accounting-software complexity.

## Scope Boundaries

- Stay within the current local-first browser app unless the user asks for more.
- Do not add auth, cloud sync, social features, gamification, or extra health metrics by default.
- Keep the current data model compatible with later sync or mobile packaging, but do not add those dependencies now.

## Stack And Conventions

- Use React with Vite in JavaScript only. Do not introduce TypeScript.
- Use Tailwind CSS v4 and preserve the existing shadcn/ui styling direction.
- Use Zustand for app state, Dexie for IndexedDB persistence, and Recharts for charts.
- Use the `@` alias for imports from `src`.
- Keep the PWA setup working with `vite-plugin-pwa` and the GitHub Pages base-path handling in `vite.config.js`.

## Current Architecture

- Persistence and entry normalization live in `src/lib/db.js`.
- App state and selectors live in `src/store/useAppStore.js` and `src/store/app-store-slices.js`.
- App view-model assembly lives in `src/hooks/useAppViewModel.js` and `src/hooks/app-view-model-sections.js`.
- App shell navigation state lives in `src/hooks/useAppShellState.js`.
- `src/components/app/AppContent.jsx` lazy-loads the Dashboard, History, Weekly Averages, Monthly Averages, About, and Settings pages.
- Shared touch-first numeric entry lives in `src/components/app/TouchNumberInput.jsx` with reusable ranges, presets, and step config in `src/components/app/touch-number-input-config.js`.
- Shared settings drawer framing lives in `src/components/app/SettingsDrawerFrame.jsx`.
- Shared dashboard calculations live in `src/lib/metrics.js` plus focused derivation modules in `src/lib/dashboard-section-metrics.js`, `src/lib/weight-trend-metrics.js`, `src/lib/consistency-metrics.js`, `src/lib/goal-progress-metrics.js`, and `src/lib/health-journey-metrics.js`.
- Dashboard rendering components should stay thin: renderers in `src/components/app/*.jsx`, display-only mapping helpers in the matching `*.helpers.js` files.
- Average summaries are built on `src/components/app/AverageSummaryPage.jsx` and surfaced through weekly and monthly page wrappers.
- CSV import and export helpers live in `src/lib/daily-log-csv.js`.
- Shared test fixtures belong in focused files under `src/test/fixtures` and should continue to be re-exported through `src/test/leanlog-test-fixtures.js`.

## Implementation Guidance

- Prefer small, local refactors over broad rewrites.
- Keep React components thin and move derivation into focused lib modules when logic grows.
- Treat missing logs as normal. Charts, summaries, and derived metrics must tolerate blanks without throwing or shaming the user.
- Preserve the current dashboard structure unless a broader product change is explicitly requested.
- Preserve CSV import as a date-merge flow and recalculate derived fields such as `weight7dma` after imports or entry edits.
- Preserve the current touch-first numeric model: stepper buttons, preset chips, mobile sliders, and read-only coarse-pointer numeric fields should stay behaviorally consistent across daily log and tracking defaults.
- Preserve discard semantics in history flows: confirming discard should restore the current draft baseline before closing or switching dates.
- Preserve local-first performance, responsive layouts, and the existing minimal visual language.
- When extending tests, favor focused lib tests for derivation logic and focused component tests for rendering and composition boundaries.

## Validation

- Run `npm test`, `npm run lint`, and `npm run build` after meaningful changes when feasible.

---
> Source: [lumachroma/leanlog](https://github.com/lumachroma/leanlog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-06 -->
