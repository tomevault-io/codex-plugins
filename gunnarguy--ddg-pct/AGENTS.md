# DDG-PCT · Copilot Instructions

## Architecture overview

Mission-control web app for a Burney Falls→Castle Crags PCT section hike. Active code lives in `pct-hike-viz/`—a **Vite + React 19 + MapLibre/Deck.gl** stack.

**Data flow:**

1. `Original-DDG-PCT-PDF.txt` + `PCT-prep-guide.md` → canonical narrative (edit these first)
2. Garmin GPX files + `scripts/` → `src/hike_data.json` (48k lines, generated—don't hand-edit coords)
3. `src/data/planContent.js` + `resourcesIndex.js` → UI copy, gear, permits, checklists
4. `App.jsx` orchestrates loading → `TrailMap`, `Sidebar`, `ElevationProfile` render

## Essential commands

```bash
# Dev server (from repo root or pct-hike-viz/)
npm run dev:viz          # binds 0.0.0.0:4173 for remote preview
cd pct-hike-viz && npm run dev   # localhost:5173

# Build + lint
npm run build:viz
cd pct-hike-viz && npm run lint

# Data pipeline
npm run fetch:pct                       # USFS ArcGIS → public/data/pct_route.geojson
python scripts/snap_camps_to_route.py   # snaps camp coords onto route
```

**Cache invalidation:** After modifying `hike_data.json`, bump `VITE_HIKE_DATA_VERSION` in `.env`.

## Component patterns

| Pattern           | Implementation                                                                    |
| ----------------- | --------------------------------------------------------------------------------- |
| Lazy loading      | `TrailMap` via `React.lazy()` so Sidebar renders immediately                      |
| Heavy memoization | `useMemo` for `campPoints`, `routeSegments`, `deckLayers` (48k JSON is expensive) |
| PropTypes         | Required on all components; match existing shapes when extending                  |
| Emoji markers     | Lightweight div markers (⛺💧📡); add types in `getTransportIcon()`               |

## Content editing

| Content                             | File                           | Notes                                 |
| ----------------------------------- | ------------------------------ | ------------------------------------- |
| UI copy (schedules, risks, permits) | `src/data/planContent.js`      | Sync with `PCT-prep-guide.md`         |
| External links + gear citations     | `src/data/resourcesIndex.js`   | Items reference `sourceIds` from here |
| Route coordinates                   | `src/hike_data.json`           | Use snapper script only               |
| Cell coverage zones                 | `src/data/connectivityData.js` | 9 checkpoints with carrier ratings    |

## Map conventions

- Basemaps from `nst-guide/osm-liberty-topo` (no API keys)
- Deck.gl `PathLayer` draws trails; extend `driveSegments`/`transport`/`waterSources` arrays in `hike_data.json` rather than scattering markers
- Transport markers: add new `type` keys + icons in `getTransportIcon()`

## File structure

```
pct-hike-viz/src/
├── App.jsx              # Data loader + layout orchestration
├── hike_data.json       # Generated geo (DO NOT hand-edit coords)
├── components/
│   ├── TrailMap.jsx         # react-map-gl + Deck.gl
│   ├── Sidebar.jsx          # 6-tab mission control
│   ├── GearPlanner.jsx      # RPG-style loadout (uses resourcesIndex)
│   └── ElevationProfile.jsx # d3-scale chart with map hover sync
├── data/
│   ├── planContent.js       # All narrative content
│   ├── resourcesIndex.js    # Citation lookup
│   └── connectivityData.js  # Cell/satellite zones
└── services/
    └── liveSatelliteService.js  # Apple satellite polling
```

## Sharp edges

- Offline dev shows blank map tiles (remote-hosted styles)
- `hike_data.json` diffs are huge—commit with "regenerated via script" notes
- `ElevationProfile` expects `hikingTrail[i][2]` for elevation; missing data renders empty
- `pct-hike-planner/` is a placeholder—no shipped code there yet

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Gunnarguy)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Gunnarguy)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
