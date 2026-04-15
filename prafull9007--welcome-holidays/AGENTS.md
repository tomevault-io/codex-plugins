<!-- Copilot instructions for the `welcome-holidays` static site -->
# Project Overview

This is a tiny static frontend app (no build step). It shows month-specific holidays from an in-memory data structure and renders them into the DOM.

- Entry points: `index.html` (markup + canvas for optional chart), `app.js` (data + rendering logic), `styles.css` (visual styling).
- No package.json / server code — preview by opening `index.html` in a browser or using a static file server.

- All submit buttons should be pink

# Big picture & structure

- `app.js` contains two responsibilities: (1) the `holidaysByMonth` data structure and (2) rendering logic (`renderForMonth`).
- `index.html` provides DOM targets by ID: `monthSelect`, `holMonth`, `holidayList`, and an optional chart canvas `holChart`.
- `styles.css` defines the visual system (variables, layout, card styles). Visual changes should be made here.
- Chart.js is included via CDN; the app does not currently populate `holChart` — this is an integration point for analytics/visualization.

# Conventions & important patterns

- Month indexing is 0-based (January = `0`). `holidaysByMonth` keys are month indices.
- Holiday objects use this shape: `{ name, date, label, icon }` where `date` is an ISO string (YYYY-MM-DD).
- `renderForMonth(monthIndex)` sorts entries using `new Date(a.date)` before rendering.
- DOM IDs are relied on heavily; do not rename `monthSelect`, `holMonth`, or `holidayList` without updating `app.js`.
- Script load order matters: `app.js` is loaded at the end of `body` (so `document.getElementById` runs successfully).

# Typical tasks & examples

- Add a holiday to August (index 7):

  ```js
  holidaysByMonth[7].push({ name: 'My Holiday', date: '2026-08-15', label: 'August 15, 2026', icon: '🎉' });
  ```

- If adding visualization with Chart.js, use the canvas ID `holChart` and initialize the chart after `renderForMonth` runs.
- To change the default selected month: see the initial `monthSelect.value = today.getMonth()` logic in `app.js`.

# Developer workflows

- Run locally: open `index.html` in a browser, or serve the folder with a static server (e.g., `npx http-server` or VS Code Live Server).
- No tests or build steps exist in the repo; keep changes minimal and self-contained.

# Guidance for AI agents

- When changing behavior prefer edits to `app.js`. Keep data and rendering colocated unless adding a new component.
- Preserve DOM IDs and the `holidaysByMonth` shape. Add migration comments if you rename keys.
- Avoid introducing new tooling or build steps without explicit user permission.
- If adding Chart.js code, check `index.html` already includes the CDN; reuse the existing `holChart` canvas.

# Files to inspect for context

- `index.html` — markup, IDs, and Chart.js canvas.
- `app.js` — month array, `holidaysByMonth`, `renderForMonth`, and event wiring.
- `styles.css` — CSS variables, layout and card styles.

If any section is unclear or you want additional examples (e.g., how to wire `holChart`), tell me which part to expand.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/prafull9007) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
