# Copilot Instructions

## Project snapshot

- Stack: TypeScript + Vite + vanilla DOM (no framework).
- Entry point: `src/index.ts`.
- Tests: Vitest (`src/__tests__/*.test.ts`, `jsdom` environment).
- Build output: `build/` (Vite). TypeScript output from checks/tests goes to `dist/`.

## Commands

- `npm run dev` - local development server
- `npm run build` - production build
- `npm run preview` - preview production build
- `npm run test` - `tsc && vitest run`
- `npm run typecheck` - `tsc`
- `npm run format` - Prettier
- `npm run compress:lines` - regenerate `src/map/lines.geojson`

## Repo conventions

- Keep local imports in `src/` with `.js` extensions (`import "./x.js"`).
- Prefer small pure helpers in `src/logic.ts` and keep DOM wiring in `src/index.ts`.
- Station IDs are Wikidata QIDs; line IDs are numeric strings defined in `src/lines.ts`.
- Metro and CPTM modes use separate localStorage keys (`src/state.ts`).
- Daily puzzle reset is tied to São Paulo time logic in `src/index.ts` (UTC-3 shift).
- Use `new URL("./file", import.meta.url)` for asset/data loading.

## Data and map docs

- Metro CSVs: `src/stations.csv`, `src/adjacencies.csv`, `src/interchanges.csv`.
- CPTM CSVs: `src/data/*_with_cptm.csv`.
- Map assets live in `src/map/`.
- `src/map/README.md` is the canonical guide for map-line data generation/compression.

---
> Source: [yancouto/metrodlesp](https://github.com/yancouto/metrodlesp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-13 -->
