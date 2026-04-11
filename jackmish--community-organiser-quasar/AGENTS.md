# Copilot / AI Agent Instructions

Purpose: Help AI coding agents get productive quickly in this repository (Quasar + Vue 3 + TypeScript app with Electron & Capacitor targets).

1. Big picture

- Frontend: Quasar (Vue 3 + Vite) in `src/`.
- Desktop: Electron integration lives in `src-electron/` and `electron-main.ts`.
- Mobile: Capacitor config and Android targets are in `src-capacitor/`.
- Domain separation: Domain-specific Vue components are co-located with their module under `src/modules/<domain>/components/` (e.g. `src/modules/task/components/`, `src/modules/group/components/`). Shared/cross-cutting components (ui, time, settings) remain in `src/components/`. Business logic and feature modules live under `src/modules/` (e.g. `task`, `group`, `day-organiser`).

2. Key files and services to inspect first

- `package.json` — primary scripts and dependencies (dev, electron, build, build:electron, android, test, test:unit, test:e2e).
- `quasar.config.ts` — project targets and build-time config.
- `src/services/appService.ts` — central app-level API surface used across components.
- `src/composables/` — composition helpers (common patterns: `useClock.ts`, `useLongPress.ts`).
- `utils/logger.ts` and `utils/storageUtils.ts` — logging and storage helpers used throughout the app.

3. Developer workflows (concrete commands)

- Install deps: `npm install`
- Dev (browser): `npm run dev` (Quasar dev server).
- Electron desktop dev: `npm run electron` (Quasar dev in electron mode).
- Build production (web): `npm run build`.
- Build Electron: `npm run build:electron`.
- Build Android (Capacitor): `npm run android`.
- Unit tests: `npm run test:unit` (Vitest). Note: common test setup in `tests/setup.ts`.
- E2E tests: `npm run test:e2e` (Playwright).

4. Project-specific conventions & patterns

- Domain grouping: domain-specific components live in `src/modules/<domain>/components/` (e.g. `src/modules/task/components/AddTaskForm.vue`); shared/cross-cutting components stay in `src/components/` (e.g. `ui/`, `time/`, `settings/`) — follow this when adding UI.
- Composition API + TypeScript: prefer composables for reusable logic; avoid ad-hoc cross-component string-based emits where a service call is clearer.
- Storage: desktop data saved under `%APPDATA%/community-organiser/` — for Electron-only changes consider `src-electron` codepaths.
- Logging: use `utils/logger.ts` helper to keep logs consistent.

---

-User instructions section (for AI agents):

- After making code changes, automatically run the following checks **without asking**: `npm run lint`, `npx tsc --noEmit`, `node scripts/check-imports.cjs`, and `npm run test:unit`. If any step fails, fix the failures and re-run the checks until the test suite passes. Do **not** run `npm run format` (formatting can create noisy diffs and may conflict with local editor settings).
- When the changes are complete and tests pass, remove or silence ordinary `console.log` output so the console remains clean; keep error, warning and important informational messages visible.
- Try to update this `copilot-instructions.md` file after significant refactorings once the refactor is implemented and its tests pass.
- If you discover a better, more reliable way to enforce or communicate these policies to the agent (for example, a pre-commit hook, CI job, or a dedicated checklist file), adopt that approach and add a short note here linking to it.
- instead of vue emits try tu utilize functions in CentralController usually CC.task.someFunction(...) or CC.group.someFunction(...) to trigger cross-component behavior. If the function doesn't exist, consider adding it to the relevant module (task/group) and then calling it from components instead of emitting events. Local functions should stay inside component or utils if something looks reusable.
- try to make at least unit tests for any new logic you add, and if you refactor something, try to add tests for the new code and keep the old tests passing. If you need to mock Electron or Capacitor APIs, use `tests/setup.ts` for common mocks.
- at this moment project structure is going to be like:
  CentralController (CC) is orchestrating other controllers, making some singleton/index solving some chaotic AI generation problems, also it's trying to avoid emit/event-listener based chaos. In the future it would gather more control.
  **CC Registry pattern (April 2026):** CC uses a typed `CCRegistry` in `src/CCRegistry.ts`. Domain wiring lives in `src/CCAccess.ts`. To add a new domain controller: (1) implement `Controllable` (`src/types/Controllable.ts`), (2) add one `.register()` line in `CCAccess.ts`. `CCReg.boot()` (called from `boot/pinia.ts` after Pinia is active) automatically wires all storage ports and lifecycle hooks — no other bootstrap changes needed. Storage receives data via `StorageController.connect(port)` instead of constructor injection.
  **CC split into CC + CCRegistry (April 2026):** `CCAccess.ts` now exports two things: `CC` (typed access proxy — default export used by components) and `CCReg` (the `CCRegistry` instance — named export used for boot and optional external registration). Each domain is registered in a single fluent line (e.g. `.register('group', GroupStoreController)`). Adding a new domain = one `.register()` line and one getter; everything else is automatic. The registry engine lives in `src/CCRegistry.ts`. `CCReg.boot()` is called from `boot/pinia.ts` after Pinia is active. `CentralController.ts` has been removed — the canonical import path is `src/CCAccess`.
- try to avoid names like Service or App. The `*Manager` naming convention has been refactored to `*Repository` (e.g., `TaskRepository`, `GroupRepository`). Prefer `*Repository` for data-layer classes/modules going forward. `Manager` was acceptable but is now superseded by this refactor (completed April 2026).
- avoid also Store name for other modules than storage. It would be confusing with storage module. It could make some layer of model/class but not core name for Controller file.
- try to use model classes, and implement some model interface. At this moment there is no model interface, but should be created at some point. For example Group model should implement Model Interface. Don't create interfaces for everything, every model separately is not needed, but if you see that there is some common logic between models, consider creating an interface for them.
- general model.ts interface should be adapted to project needs, not overcomplicated. It should be similar to Laravel backend models, but adapted to frontend needs. It should have some common properties like id, created_at, updated_at, and some common methods like save, delete, etc. But it should not be overcomplicated with too many methods or properties that are not needed at this moment. It should be simple and easy to use. I need to investigate Pinia - how could help with models. Currently its installed but probably its not used in any reasonable way.
- there is module called day-organizer- needs to be refactored. You can treat it as app folder or core folder - temporary.
- there is a need to use list object similar to Laravel Collections. Remember that when you are trying to filter some list of tasks or groups. It would be nice moment to shorten code, refactor it. Maybe there is already something ready to apply.

5. Testing notes

- Unit tests in `tests/unit/` use Vitest. Mock filesystem or platform-specific APIs when testing Electron/Capacitor features.
- E2E tests in `tests/e2e/` use Playwright; the app should be in a testable dev mode for reliable e2e runs.

PowerShell check for Vue SFC template errors

- Quick PowerShell scan to detect Vue single-file components containing more than one `<template>` (useful to catch plugin:vite:vue SFC errors from the terminal):

```powershell
# Run from repository root (Windows PowerShell / PowerShell Core)
$files = Get-ChildItem -Recurse -Filter *.vue src
$bad = @()
foreach ($f in $files) {
  $n = (Select-String -Path $f.FullName -Pattern '<template' -AllMatches).Matches.Count
  if ($n -gt 1) { $bad += "$($f.FullName): $n templates" }
}
if ($bad.Count -gt 0) {
  $bad -join "`n"; exit 1
} else {
  Write-Output "No multi-template SFCs found."; exit 0
}
```

- Use this before running `npm run dev` or `npm run build` to quickly detect offending `.vue` files without starting Vite. You can add this script to CI or run it locally in PowerShell.

6. Common pitfalls for automation

- Many components were initially generated by AI and may duplicate logic; prefer refactoring to central composables or services rather than copy-pasting.
- Watch for inconsistent naming of imports across files — use project-level exports (services or modules) to reduce fragility.

7. Where to look for examples

- UI pattern: `src/modules/task/components/AddTaskForm.vue` (task creation patterns).
- Domain module: `src/modules/group/` and `src/modules/day-organiser/` show how business logic is organized.
- Electron entry: `src-electron/electron-main.ts`.

8. PR checklist for AI edits

- Run `npm run format` and `npm run lint` locally.
- Run unit tests: `npm run test:unit` and fix failures.
- Avoid introducing new global side-effects; prefer adding to `src/services/` or `src/composables/`.

If anything here is unclear or you want the file expanded with examples/quick links to specific lines, tell me which area to expand. 9. Examples & preferred patterns (short, copyable)

- Use the central `appService` for cross-cutting APIs — avoid creating ad-hoc global emits.

```ts
import { app } from 'src/services/appService';

const storage = app('storage');
await storage?.save('groups', groupsData);
```

- Register long-lived services in a module root (see `src/modules/day-organiser/apiRoot.ts`):

```ts
import { registerAppService } from 'src/services/appService';
registerAppService('storage', storageApi);
```

- Prefer composables for reusable logic in `src/composables/`.
  Example: `useLongPress.ts` provides the press behavior used by several components.

- Component → module pattern: components call public module APIs exported from `src/modules/*` rather than reaching into deep module internals.
  Example: `import { task } from 'src/modules/day-organiser/apiRoot';`

- Tests: put unit tests under `tests/unit/` and use `tests/setup.ts` for common mocks. For Electron/Capacitor logic mock platform APIs.

- Storage path (desktop/Electron): expect data under `%APPDATA%/community-organiser/` — use `src/modules/storage` or `app('storage')` when adding persistence.

- Logging: use `utils/logger.ts` for consistent output. Prefer `logger.info()` / `logger.error()` over console.log.

- Import style: the project commonly imports from `src/...` (Vite/Quasar alias). If uncertain, mirror nearby files' imports.

- Avoid: adding new global event channels with string names. If you need cross-component communication, either add a composable, a service under `src/services/`, or extend `appService`.

10. Where to put longer style rules

- Keep this file short. Add longer examples, linting rules, and component style in `CONTRIBUTING.md` or `docs/CODING.md` and link from this file.

- Styling guideline (grayscale & hover): If you are asked to change grayscale, change only the grayscale/opacity values — do not add other visual effects. Specifically avoid hover styles that move, scale, or shift text or primary interactive elements (these can blur text and invalidate the hover/tap area). Transform/scale operations are allowed only for additional decorative elements (for example an underline or outline), and color changes on hover are acceptable. Avoid moving borders or changing layout on hover.

If you want, I can now add 6–10 concrete code examples to `docs/CODING.md` and add links to specific lines in `src/services/appService.ts` and `src/modules/day-organiser/apiRoot.ts`.

11. Avoid monoliths & deduplication guidance

- Keep files focused and small: prefer many small modules (single responsibility) over one large file. When a `.vue`, `.ts`, or `.js` file grows past ~300 lines, consider splitting logic into a composable (`src/composables/`) or a service (`src/services/`).
- Move shared logic to composables or services rather than duplicating it across components. Example candidates: date/time helpers, long-press logic, storage access, and logging.
- Before adding new code, search the repo for similar implementations (keywords, helper names, or behavior). If a match exists, prefer reuse:

```bash
# Example: find existing storage usage or helpers
rg "app\('storage'\)|useLongPress|save\(" src || true
```

- When refactoring duplicated code, create a small adapter in the original place to preserve call sites, then migrate callers to the new composable/service in follow-up commits.
- If you must create a new helper, add it under `src/composables/` (UI-centric) or `src/services/` (cross-cutting), and document the intent in a short comment and in this file.

12. Quick functionality map (where things live)

- Day organiser / domain root: `src/modules/day-organiser/apiRoot.ts` — exposes `task`, `group`, `storage` APIs for components to import.
- Task domain: `src/modules/task/` — task business logic and public API used by UI components.
- Group domain: `src/modules/group/` — group hierarchy, persistence calls typically via `app('storage')`.
- Storage: `src/modules/storage/` and `src/services/appService.ts` (storage is registered as `storage` service in `apiRoot.ts`).
- UI components: domain-specific components in `src/modules/<domain>/components/` (e.g. `task/`, `group/`); shared components in `src/components/` (e.g. `ui/`, `time/`, `settings/`).
- Composables: `src/composables/` — reusable UI and behavior code (e.g., `useClock.ts`, `useLongPress.ts`).
- Services: `src/services/` — cross-cutting services (notably `appService.ts`).
- Electron entry & desktop-specific: `src-electron/electron-main.ts` and `src-electron/*`.
- Capacitor / mobile packaging: `src-capacitor/` (Android config and web assets).
- Utilities: `src/utils/logger.ts`, `src/utils/storageUtils.ts` — prefer these instead of ad-hoc console/storage logic.

13. Workflow checklist for de-duplication tasks

- Search for existing code with `rg`/`rg`-style searches before adding new helpers.
- If you add a helper, add a short unit test in `tests/unit/` and a note in this file linking to the new helper.
- When consolidating duplicates, create small commits: (1) add new helper, (2) migrate callers, (3) remove old duplicates — this makes review easier.

---

If you'd like, I can now create `docs/CODING.md` with expanded examples and add a small checklist/template for refactors and deduplication PRs.

Note for AI agents: Prefer the modern GroupActive API (`set`, `setById`, `selectAll`) instead of calling a non-standard `activate(...)` method. If you encounter code using `activate(...)`, update the call-sites to use `set(...)` or `setById(...)` rather than adding compatibility aliases.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jackmish)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/jackmish)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
