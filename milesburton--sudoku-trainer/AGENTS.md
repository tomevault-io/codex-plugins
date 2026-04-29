<!-- .github/copilot-instructions.md -->
Purpose: Help AI coding agents become productive in this Angular + NgRx example app.

- Quick commands:
  - Install deps: `npm install`
  - Dev server: `npm start` (runs `ng serve`)
  - Build: `npm run build` (uses Angular CLI)
  - Run tests: `npm test` (Vitest, config in `vitest.config.ts`)
  - Test watch: `npm run test:watch`
  - Coverage (local): `npm run coverage` → generates `coverage/` (text, lcov, html)
  - Coverage (CI): `npm run test:ci` → runs tests once and enforces thresholds

- Big picture:
  - Small Angular application located under `src/` with a single feature: a Warning Centre UI.
  - App bootstraps `AppModule` (`src/main.ts` → `src/app/app.module.ts`).
  - Global state is NgRx Store. The `warnings` feature is implemented in `src/app/warnings/`.

- State & data-flow patterns (important to follow):
  - Feature key: `warningsFeatureKey` is declared in `src/app/warnings/warnings.reducer.ts` and registered in `AppModule` via `StoreModule.forRoot({ [warningsFeatureKey]: warningsReducer })`.
  - Actions: use `WarningsActions` from `src/app/warnings/warnings.actions.ts` (created with `createActionGroup`).
    - Example dispatch: `this.store.dispatch(WarningsActions.removeMessage({ id }))` (see `WarningCentreComponent`).
  - Selectors: prefer `store.select(selector)` and keep selectors in `src/app/warnings/warnings.selectors.ts`.
    - Example use: `this.store.select(selectAllMessages)` → exposes `messages$` in the component.
  - Reducer seeds: `initialState` in `warnings.reducer.ts` contains `seededMessages` used for dev & tests — tests may rely on these seeded values.

- Component & styling conventions:
  - Components use `ChangeDetectionStrategy.OnPush` where appropriate (`WarningCentreComponent`).
  - Observable properties end with `$` (e.g. `messages$`, `counts$`, `borderClass$`). Use `async` pipe in templates.
  - UI primitives come from PrimeNG — check `warning-centre.module.ts` for imported Prime modules (`ButtonModule`, `OverlayPanelModule`, `DropdownModule`).

- Testing specifics:
  - Tests run with Vitest (`vitest.config.ts`). The test environment is `jsdom` and `src/test-setup.ts` sets up Angular testing with `zone.js`.
  - Use the existing spec file pattern in `src/app/warning-centre/warning-centre.component.spec.ts` as an example for component tests.
  - Coverage: Vitest coverage is configured in `vitest.config.ts` with V8 provider. Reports are emitted to `coverage/` as `text`, `lcov`, and `html` and global thresholds are enforced (adjust thresholds in the config if needed).

- Files to inspect first when working on features or bugs:
  - UI / components: `src/app/warning-centre/` (module, component, template, styles, spec)
  - State: `src/app/warnings/` (model, actions, reducer, selectors)
  - App root: `src/app/app.module.ts`, `src/main.ts`
  - Test config: `vitest.config.ts`, `src/test-setup.ts`

- Common tasks & tips for edits:
  - Adding a new warning-related UI: add feature code under `src/app/warnings/` and register only the reducer feature key in `AppModule` if it must be global.
  - When altering selectors or reducer shapes, update all components that `select(...)` the feature (search for `selectWarningsState` / `selectAllMessages`).
  - Keep Observable naming (`$` suffix) and OnPush strategy to avoid change-detection issues when returning new arrays/objects from the reducer.

- Integration and external deps:
  - UI: `primeng` + `primeicons` (check `warning-centre.module.ts`).
  - State: `@ngrx/store` and `@ngrx/store-devtools` (devtools configured in `AppModule`).
  - Test tooling: `vitest`, `@analogjs/vitest-angular` and `jsdom`.

If anything in these notes is unclear or you'd like more examples (dispatching actions, writing a selector, or a test scaffold), tell me which area to expand and I will iterate.

---
> Source: [milesburton/sudoku-trainer](https://github.com/milesburton/sudoku-trainer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-29 -->
