# Copilot / AI Agent Instructions for flood-frontend

## Quick project summary ✅
- Small React single-page app that visualizes Delhi wards and shows flood risk.
- Major pieces: UI shell in `src/App.js`, map rendering in `src/components/MapView.jsx`, alerts UI in `src/components/ActionPanel.jsx`, risk logic in `src/utils/riskEngine.js`, and static geo data in `src/data/delhi_wards.geojson`.
- Built with CRA (react-scripts) and uses `react-leaflet` + OSM tiles for map tiles.

---

## How to run / build 🔧
- Install dependencies: `npm install` (project uses `react-scripts`).
- Start dev server: `npm start` (runs `react-scripts start`).
- Create a production build: `npm run build`.
- Typical issues: Leaflet requires `leaflet/dist/leaflet.css` (already imported in `src/index.js`) and the map container must have a non-zero height (see `MapContainer` style in `MapView.jsx`).

---

## Files to inspect / change (concrete examples) 🔎
- `src/components/MapView.jsx`
  - Imports geojson as `wards` and iterates features via `onEachFeature`.
  - Uses `generateRisk(rainfall, timeHorizon)` (from `utils/riskEngine.js`) to style each ward.
  - Binds HTML popup strings to layers (not React components).
- `src/utils/riskEngine.js`
  - Exports `generateRisk(rainfall, timeHorizon)` returning `{score, level, confidence, action}`.
  - Uses `Math.random()`; therefore risk is non-deterministic across renders and features.
- `src/App.js`
  - Controls top-level state: `rainfall`, `timeHorizon`, `alerts`.
  - `handleAlert` dedupes alerts: `setAlerts(prev => prev.includes(alert) ? prev : [...prev, alert])`.
- `src/data/delhi_wards.geojson`
  - Edit here to change displayed wards or to add ward-level attributes (e.g., `population`, `elevation`).

---

## Project-specific patterns & conventions ⚙️
- Functional components with local state (React hooks). No TypeScript or PropTypes present.
- Inline styles are used for layout (look at `App.js` and `ActionPanel.jsx`).
- GeoJSON is imported as a static module rather than fetched at runtime.
- Popups use string templates (HTML snippets), not React `Popup` components.
- Alerts are kept in a simple deduplicated array in `App.js` and displayed by `ActionPanel.jsx`.

---

## Integration & external dependencies 🔗
- `react-leaflet` (map components) and `leaflet` (map engine and CSS).
- OpenStreetMap tile server: `https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png` (no API key required).

---

## Known gotchas & places an agent should pay attention ⚠️
- `generateRisk` is feature-agnostic: it does not use `feature.properties`. Adding ward-specific inputs will require changing where and how the function is called.
- Because `generateRisk` uses randomness, map renders are not reproducible; replace `Math.random()` with a seeded RNG for deterministic testing.
- `onEachFeature` calls `generateRisk` for every feature on each render — with large geojson this can be expensive. Consider memoization keyed by `(feature.id, rainfall, timeHorizon)`.
- Popups use HTML strings; if you need interactive React-based popups, migrate to react-leaflet's `<Popup>` and controlled components.

---

## Good first tasks for an AI agent 👶
- Add unit tests for `generateRisk` (extract deterministic branches and test thresholds).
- Make risk deterministic for CI by optionally injecting a RNG seed.
- Read per-ward attributes from `feature.properties` to compute ward-specific risk.
- Replace HTML-string popups with React `<Popup>` components where richer UI is needed.

---

## Contribution / PR guidance 🧭
- Keep changes small and focused (this repo is intentionally minimal).
- If adding dependencies, prefer lightweight libs (bundle size matters for frontends).
- Run `npm run build` locally before submitting a PR to verify bundle builds.

---

If anything above is unclear or you want me to expand any section (for example add example unit tests, or a short task list for introducing determinism to `generateRisk`), tell me which section and I will iterate. 👇

---
> Source: [ayush678-hub/FloodSense](https://github.com/ayush678-hub/FloodSense) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-29 -->
