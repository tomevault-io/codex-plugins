# Copilot Instructions – weekplan (Wochen-Power)

Purpose: Guide AI agents contributing to this vanilla JS, no-build, browser-only week planner with gamification. Keep changes small, defensive, accessible, and consistent with existing patterns.

## Architecture Summary

- Entry bootstrap: `js/main.js` (hydrate saved data -> render UI -> init events/games -> subscribe -> cleanup -> notifications).
- Central reactive state: `js/state.js` (plain object + `updateState`, `subscribe`, guarded by `sanitizeStateUpdate`, deep-clone reader `getState`). Never mutate `state` directly; always call `updateState(partial)` with validated shapes.
- Persistence: `js/storage.js` saves selected keys (`tasks`, `pcStundenGesamt`, `wochenZiel`, `theme`, `coins`) via debounced subscribers in `main.js`.
- Tasks domain: `js/tasks.js` (CRUD + validation + streak & points + limited field whitelist via `sanitizeTaskData`). UI rendering logic lives in `ui.js` — keep domain logic here, presentation there.
- UI rendering: `js/ui.js` builds all DOM fragments imperatively (no framework). Diff strategy: `updateTasksUI` compares previous vs new tasks to patch DOM. Adding new widget? Follow pattern: structure render + update function + integrate into `renderAllUI` & `updateAllTrackers` if it tracks state.
- Events & delegation: `js/events.js` single body click handler routes actions (`handleTaskCardActions`, etc.). Prefer extending these handlers over adding scattered listeners.
- Gamification: `js/games.js` (memory + quiz), `js/effects.js` (confetti/ripple), `js/audio.js` (dynamic Tone.js load, guarded init), coins + achievements (`config.js`, used by `ui.js`).
- Theming: `js/theme.js` normalizes theme IDs, updates DOM classes, updates meta color and page title (progress percentage). Theme-specific CSS classes: `theme-<id>`.
- Sharing/printing: `js/share.js` (export JSON + print with dynamic title based on next task).

## Data & State Conventions

- Task object fields: `{ id, name, kategorie, date (YYYY-MM-DD, local), durationInMinutes, erledigt }` — only these fields persist. Use `sanitizeTaskData` for merges. IDs use `task-${Date.now()}`.
- Dates: Always local day form via helpers: `getISODate`, `parseLocalISODate`, `getStartOfWeek`. Do not construct bare `new Date('YYYY-MM-DD')` for logic; use helpers.
- PC time limit & weekly goal numeric; validate non-negative; store as numbers.
- State keys guarded: do not introduce prototype-polluting keys; keep additions minimal and documented.

## UI / Accessibility Patterns

- ARIA: Buttons controlling menus set `aria-expanded`; theme options use `aria-checked`; task errors set `aria-invalid` and update inline error nodes.
- Modals: Use `showModalElement` / `hideModalElement`; body class `modal-open` toggled centrally. Do not manually manipulate scroll locking.
- Avoid `innerHTML` for user-provided text (use `textContent`). When dynamic SVG markup inserted, ensure it is static & trusted.

## Async / External Loading

- SortableJS + Tone.js loaded dynamically with fallbacks and timeouts; always null-check external APIs (`Sortable`, `Tone`, synth methods). If extending audio, follow `initSounds` pattern (dispose old, create new, update state atomically).

## Adding Features – Checklist

1. Define pure domain logic (e.g., scoring, filtering) outside UI modules when reusable.
2. Extend state via `updateState({ newKey })`; add default in initial `state` object and ensure persistence only if needed (update `storage.js`).
3. Add render structure + update function; call from `renderAllUI` and/or specialized aggregators like `updateAllTrackers`.
4. Integrate events by enhancing existing delegation in `events.js` instead of per-element listeners.
5. Write tests (Node test runner) for pure functions (utilities, task logic) in `tests/*.test.mjs` (see existing patterns like `parseLocalISODate.test.mjs`). For DOM-manipulating code, structure logic so pure pieces stay testable.

## Testing & Scripts

- Unit tests: `npm test` (Node built-in test runner, ESM). Use `.test.mjs` extension.
- Functional subsets: `npm run test:functions`, `npm run test:ui`.
- E2E: `npm run test:e2e` (Playwright). CI variant installs deps first.
- Avoid introducing build steps; prefer CDN or dynamic import if a lightweight lib is truly required.

## Defensive Coding Rules

- Always guard array accesses, treat possibly undefined objects defensively (`Array.isArray(...) ? ... : []`). Existing code heavily validates — match style.
- When updating arrays (tasks), create new arrays; never mutate in place if subscribers depend on identity changes.
- Preserve non-serializable fields when cloning state (follow `getState` pattern; do not add uncloneable objects outside known exceptions `sounds`, callbacks).

## Performance Considerations

- Re-render granularity: rely on `updateTasksUI` diff instead of full `renderPlan` except on initial load/week change.
- Debounce persistence (`saveData`) — if adding frequent-updating state, reuse that debounce or create a similar one.

## Internationalization / Language

- UI language currently German; maintain wording style (short, motivational). New labels follow existing case & emoji style.

## When Unsure

- Prefer minimal, observable change with a test covering new logic.
- Follow existing naming: camelCase for functions, SCREAMING_SNAKE for constants.
- Keep file length manageable; if a module exceeds ~800–900 lines (e.g., `ui.js` nearing that), consider extracting cohesive submodule (e.g., achievements, trackers) but keep public surface backward compatible.

Provide diffs only for changed files. Ask before introducing new dependencies.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DaFum)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/DaFum)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
