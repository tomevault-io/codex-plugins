# webmet25 — Copilot Instructions

## 🧠 Context & Knowledge Base
> **CRITICAL:** You are operating in a multi-root workspace alongside the `radarlib` repository. 
> Before writing code, route your knowledge by reading the appropriate documentation:

- `docs/DISCOVERY_REPORT.md`: Full technical analysis, DB schema, API endpoints, and identified risks.
- `README.md` (root): High-level purpose, tech stack, and architecture overview.
- `docs/DATA_FLOW.md`: How `webmet25` ingests, processes, and displays `radarlib` data. **Read this before changing the indexer or API.**
- `docs/COMPONENTS.md`: Breakdown of UI/frontend modules. **Read this before making frontend/Leaflet changes.**

---

## About This Repository
webmet25 is the **data consumer** in the radarmet system. It is one 
of two repositories. It ingests Cloud-Optimized GeoTIFF (COG) files 
produced by **radarlib**, indexes them into a PostgreSQL/PostGIS 
database, and serves them via a REST API and interactive Leaflet 
frontend.

---

## System Context
```text
radarlib (producer)
    │
    ├── outputs GeoTIFF COGs to ROOT_RADAR_PRODUCTS_PATH
    ├── outputs GeoJSON tops & cores to TOPS_AND_CORES_DIR
    │
    ▼
webmet25 (consumer)
    │
    ├── Indexer watches ROOT_RADAR_PRODUCTS_PATH
    ├── TopsAndCoresWatcher watches TOPS_AND_CORES_DIR
    │   ├── parses filenames
    │   ├── extracts metadata
    │   └── stores in PostgreSQL/PostGIS
    │
    ├── FastAPI serves metadata + tiles
    │
    └── Leaflet frontend renders radar map
```

### radarlib Output Contract (what we depend on)
> ⚠️ webmet25 depends entirely on radarlib's output format.
> Never assume a different format without checking radarlib's 
> `docs/radarlib_EN.md` Output Contract section first.
> ⚠️ This contract is sourced from radarlib. If radarlib 
> changes its output format, update this section immediately.
> In case we are working on a multi-root workspace, you can directly inspect `radarlib` code to verify this.

### Primary Output Format
- **GeoTIFF (COG):** This is the primary and current output format.
  Cloud-Optimized GeoTIFF is the production standard.
- **PNG:** Deprecated. Kept only for backward compatibility.
  Do not build new features around PNG output.

### File Naming Convention

#### Current Production Format (Pattern 0)
`<RADAR_NAME>_<STRATEGY>_<VOL_NR>_<TIMESTAMP>_<FIELD>[o].<ext>`

| Token | Description | Example |
|-------|-------------|---------|
| `RADAR_NAME` | Radar station identifier | `RMA1` |
| `STRATEGY` | 4-digit scanning strategy code | `0315` |
| `VOL_NR` | 2-digit volume number | `01`, `02`, `04` |
| `TIMESTAMP` | ISO 8601 format: `YYYYMMDDTHHMMSSZ` | `20260401T205000Z` |
| `FIELD` | Radar field/variable name | `ZDR`, `DBZH` |
| `[o]` | Letter `o` suffix = raw/non-filtered data. Absent = filtered data | `ZDRo` vs `ZDR` |
| `ext` | File extension | `tif` (primary), `png` (deprecated) |

Examples:
```
RMA1_0315_01_20260401T205000Z_DBZH.tif     # filtered reflectivity, vol 01
RMA1_0315_01_20260401T205000Z_DBZHo.tif    # unfiltered (raw) reflectivity, vol 01
RMA1_0315_04_20260401T205000Z_COLMAX.tif   # column max, vol 04 (vigilant)
```

#### Legacy Format (Pattern 1 — backward compatibility only)
`<RADAR_NAME>_<TIMESTAMP>_<FIELD>[o]_<ELEVATION>.<ext>`

Example: `RMA1_20260401T205000Z_ZDRo_00.tif`

Legacy files are indexed with `strategy=None` and `vol_nr=None`. The indexer
logs a WARNING for each legacy file encountered.

PNG equivalent (deprecated, backward compat only):
`RMA1_20260401T205000Z_ZDR_00.png`

### Folder Structure
```text
ROOT_RADAR_PRODUCTS_PATH/
└── <RADAR_NAME>/
    └── /YYYY/
        └── /MM/
            └── /DD/
                ├── RMA1_20260401T205000Z_ZDR_00.tif
                ├── RMA1_20260401T205000Z_ZDRo_00.tif
                └── RMA1_20260401T205000Z_ZDR_00.png ← deprecated
```

### GeoTIFF Metadata Fields

| Field | Value | Purpose |
|---|---|---|
| **CRS** | EPSG:3857 | Web Mercator (Pseudo-Mercator). Frames endpoint
|         |           | converts to WGS84 for X-Bbox-* headers. |
| **radarlib_cmap** | Colormap name string | Name of matplotlib colormap used (e.g., `"grc_th"`) |
| **vmin** | Float | Minimum data value for color scaling |
| **vmax** | Float | Maximum data value for color scaling |
| **field_name** | String | Radar field name (e.g., `"DBZH"`) |
| **timestamp** | ISO 8601 | Data acquisition timestamp |

### Critical Rules
- **Never change this contract without updating webmet25 indexer.**
- **Do not add new output formats without updating both repos.**
- When implementing multi-elevation support in the future, the
  `ELEVATION` token must remain zero-padded to 2 digits (e.g.,
  `05`, `10`) to preserve consistent file naming.
- PNG generation should not be extended or improved. If a task
  involves PNG output, flag it and ask for confirmation.

> ⚠️ webmet25 depends entirely on radarlib's output format.
> Never assume a different format without checking radarlib's 
> `docs/radarlib_EN.md` Output Contract section first.

- **Primary format:** Cloud-Optimized GeoTIFF (.tif)
- **PNG:** Deprecated, backward compatibility only
- **File naming:**

---

## Tech Stack
- **Language:** Python 3.11
- **Backend:** FastAPI, SQLAlchemy, Alembic, GeoAlchemy2, Uvicorn
- **Geospatial:** Rasterio, rio-tiler, Shapely, GDAL
- **Database:** PostgreSQL with PostGIS, Alembic for migrations
- **Frontend:** Leaflet, CartoDB basemaps, plain JavaScript ES6
- **DevOps:** Docker, Docker Compose, VSCode Dev Containers

---

## Project Architecture
api/ # FastAPI backend
database/ # Shared SQLAlchemy models
indexer/ # COG file watcher and database updater
frontend/ # Static files served via Nginx
radar_db/ # Shared Python package: DB models and utilities
docs/ # Project documentation
tests/ # Automated tests


---

## Database Schema
| Table | Primary Key | Key Fields |
|-------|-------------|------------|
| `Radar` | `code` | `title`, `center_lat`, `center_long`, `is_active` |
| `RadarProduct` | `id` | `product_key` (UNIQUE), `product_title`, `min_value`, `max_value` |
| `RadarCOG` | `id` | `radar_code` (FK), `product_id` (FK), `estrategia_code` (FK), `file_path` (UNIQUE), `observation_time`, `polarimetric_var`, `vol_nr`, `radar_coverage_m`, `status` |
| `TopsAndCores` | `id` | `radar_code` (FK), `observation_time` (indexed), `file_path` (UNIQUE), `core_count`, `top_count`, `feature_count`, `status` (COGStatus), `strategy`, `vol_nr` |
| `Reference` | `id` | `product_id` (FK), `value`, `color` |
| `Estrategia` | `code` | `description` |
| `Volumen` | `id` | `value` (integer) |

**Key RadarCOG columns added in recent versions:**
- `vol_nr` (String 16): Volume number from filename, e.g. `"01"`, `"04"`. `NULL` for legacy files.
- `estrategia_code` (FK → `Estrategia.code`): Strategy code, e.g. `"0315"`. `NULL` for legacy files.
- `radar_coverage_m` (Float): Radar coverage radius in metres, from radarlib COG tag. `NULL` for legacy files.
- `polarimetric_var` (String 16): Exact field name including `o`-suffix, e.g. `"DBZHo"`. Used for exact-match filtering in the `/cogs` endpoint.

**Key RadarProduct columns added in recent versions:**
- `default_cmap` (String 64, nullable): DB-canonical default colormap name (e.g. `"grc_th"`).
- `min_value`, `max_value`: Authoritative data range for colour scaling.

**Colormap tables:**
- `colormap_stops`: One row per channel point (`cmap_name`, `channel` r/g/b, `position`, `val_left`, `val_right`, `sort_order`, `is_system`). 8 system colormaps seeded: `grc_th`, `grc_th2`, `grc_rain`, `grc_g`, `grc_rho`, `grc_zdr`, `grc_vrad`, `Theodore16`.
- `product_colormap_options`: Many-to-many between `product_key` and `cmap_name`.

**ColormapService** (`api/app/services/colormap_service.py`): thread-safe singleton with 5-minute TTL cache. Exposes `get_cmap(name)`, `default_for_product(key)`, `options_for_product(key)`, `list_cmap_names()`, `invalidate()`. Colormap resolution order: DB → hardcoded builders in `utils/colormaps.py` → PyART → matplotlib.

**Unique constraint on RadarCOG:** `(radar_code, product_id, observation_time, elevation_angle, vol_nr)` — allows the same field from different volumes (e.g. `DBZH` vol 01 vs vol 04) to coexist as distinct records.

---

## API Contract
| Method | Path | Description |
|--------|------|-------------|
| GET | `/health` | Health check |
| GET | `/radars` | List all radars |
| GET | `/radars/{radar_code}` | Get radar details |
| GET | `/products` | List radar products |
| GET | `/cogs` | Query COG metadata. Supports `strategy` and `vol_nr` (repeatable) query params for coverage-mode filtering |
| GET | `/tiles/{cog_id}/{z}/{x}/{y}.png` | Render Web Mercator tile (v1) |
| GET | `/tiles/{cog_id}/metadata` | Get tile metadata |
| GET | `/products/{product_key}/colormap` | Get product colormap (Reference table) |
| GET | `/colormap/names` | List all DB-defined colormap names |
| GET | `/colormap/options` | Per-product colormap option lists (DB-backed, `ProductColormapOption`) |
| GET | `/colormap/defaults` | Per-product default colormap name (`RadarProduct.default_cmap`) |
| GET | `/colormap/colors/{cmap_name}` | Hex color list for a colormap |
| GET | `/colormap/info/{product_key}` | Full colormap info for a product |
| POST | `/colormap/cache/invalidate` | Flush the in-process colormap cache after DB edits |
| GET | `/frames/{cog_id}/image.png` | Full COG as single georeferenced PNG (v2). Supports `colormap`, `vmin`, `vmax`, `filter_vmin`, `filter_vmax`, `smooth`, `smooth_sigma` params. Returns bbox in `X-Bbox-*` headers |
| GET | `/tops-cores` | Query TopsAndCores metadata records by `radar_codes[]` + `time_from` + `time_to` |
| GET | `/tops-cores/{id}/features` | Fetch raw GeoJSON FeatureCollection from disk by record ID |

### `/cogs` query parameters
| Param | Type | Description |
|-------|------|-------------|
| `radar_code` | str | Filter by radar |
| `product_key` | str | Exact-match on `polarimetric_var` or product |
| `strategy` | str | Filter by volume strategy, e.g. `0315` |
| `vol_nr` | str (repeatable) | Filter by volume number(s), e.g. `?vol_nr=01&vol_nr=02` |
| `start_time` | ISO 8601 | Lower bound on `observation_time` |
| `end_time` | ISO 8601 | Upper bound on `observation_time` |
| `page` / `page_size` | int | Pagination (default 1 / 50, max 200) |

### frames endpoint response headers
- `X-Bbox-West/South/East/North` — WGS84 bounding box (reprojected from native CRS)
- `X-Width`, `X-Height` — image dimensions in pixels
- `X-Overview-Factor` — overview level used (always 1 currently)
- `Cache-Control`, `ETag` — same strategy as tile endpoint

### frames endpoint query parameters
| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `colormap` | str | from COG tag | Matplotlib colormap name |
| `vmin` / `vmax` | float | from COG tag | Color scale range |
| `filter_vmin` / `filter_vmax` | float | null | Mask pixels outside range (transparent) |
| `smooth` | bool | false | Apply Gaussian smoothing before colormap |
| `smooth_sigma` | float | 0.8 | Gaussian sigma (ignored when `smooth=false`) |

### Gaussian Smoothing
- **Implementation:** `api/app/services/smoothing.py` — `scipy.ndimage.gaussian_filter` applied to the raw float data array **before** colormap lookup.
- **Cache:** When `smooth=false` the sigma slot in the cache key is `None`, so all unsmoothed requests share the same L1/L2 entry. Smoothed variants get their own keys.
- **Frontend:** `smooth` and `smooth_sigma` are appended to the `/frames/{id}/image.png` URL by `shared/api.js`. The settings panel exposes a toggle and a sigma slider.

---

## Admin Panel & Admin API

A standalone admin SPA for CRUD over the database, served at `/admin` and backed by the `/api/v1/admin/*` routes ([`api/app/routers/admin.py`](../api/app/routers/admin.py)).

### Authentication (temporary)
- Both `/admin` (and `/admin/`) and `/api/v1/admin/` are protected by **nginx HTTP Basic Auth** (`admin.htpasswd`) — see [`frontend/nginx.conf`](../frontend/nginx.conf).
- ⚠️ This is a stopgap. The public `/api/v1/*` API remains open. **TODO: replace Basic Auth with JWT before production.** Do not build new auth assumptions on Basic Auth.

### Admin API endpoints (base `/api/v1/admin`)
| Resource | Endpoints |
|----------|-----------|
| Radars | `GET /radars`, `GET /radars/{code}`, `POST /radars`, `PUT /radars/{code}`, `PATCH /radars/{code}`, `DELETE /radars/{code}` |
| Products | `GET /products`, `GET /products/{id}`, `POST/PUT/PATCH/DELETE /products[/{id}]` |
| References | `GET /references`, `GET /references/{id}`, `POST/PUT/DELETE /references[/{id}]`, `DELETE /references?product_id=` (bulk) |
| COGs | `GET /cogs` (paginated + filters), `GET /cogs/{id}`, `PATCH /cogs/{id}` (status), `DELETE /cogs/{id}`, `DELETE /cogs?...` (bulk by filters) |
| Estrategias | `GET /estrategias`, `GET/POST/PUT/DELETE /estrategias[/{code}]` |
| Volumenes | `GET /volumenes`, `GET/POST/PUT/DELETE /volumenes[/{id}]` |
| Tops & Cores | `GET /tops-cores` (paginated), `GET /tops-cores/{id}`, `PATCH /tops-cores/{id}` (status), `DELETE /tops-cores/{id}`, `DELETE /tops-cores?...` (bulk) |
| Colormap stops | `GET /colormap-stops` (summaries: name, stop_count, is_system), `GET /colormap-stops/{cmap_name}`, `POST /colormap-stops` (single row), `DELETE /colormap-stops/{cmap_name}` (non-system only → 403) |
| Colormap (hex) | `POST /colormap-from-hex` — create from `{cmap_name, stops:[{position,color}], product_keys[]}`; **409 if name exists** (no upsert) |
| Colormap options | `GET /colormap-options[?product_key=]`, `POST /colormap-options`, `DELETE /colormap-options/{id}` |

> There is **no colormap update endpoint**. The frontend edits a colormap by **delete-then-recreate** (`DELETE /colormap-stops/{name}` → `POST /colormap-from-hex`) and reconciles product options afterward.

### Admin frontend ([`frontend/public/admin.html`](../frontend/public/admin.html), [`js/admin.js`](../frontend/public/js/admin.js), [`js/admin-api.js`](../frontend/public/js/admin-api.js), [`css/admin.css`](../frontend/public/css/admin.css))
- Hash-routed SPA (sections: `dashboard`, `radars`, `products`, `references`, `cogs`, `tops-cores`, `estrategias`, `volumenes`, `colormaps`, `colormap-options`). Section nav uses `history.replaceState` so browsing the admin never pushes browser history.
- **Modern-light theme** (`admin.css`), distinct from the dark main app. OHMC logo in the sidebar.
- **Entry / return:** the main map links to `/admin` from the **Settings panel** (`#admin-link`), setting a per-tab `sessionStorage` flag `webmet25_admin_from_main`. The admin's **← Volver al mapa** button calls `history.back()` when that flag is set (browser bfcache restores the map exactly as left — selections, frame, zoom), else navigates to `/`.
- **Filtering/sorting (Django-admin style):** every client-loaded table gets a config-driven filter bar (`FILTER_CONFIG`) = global search + per-column facets (`text` substring / `select` / `boolean`) + live result count, applied as pure DOM row show/hide (no refetch, no focus loss). All meaningful columns are sortable headers (▲/▼). COGs/Tops keep server-side filters + a quick page-search.
- **Row actions** render as inline SVG icons (pencil = edit, trash = delete), `currentColor`-tinted.
- **Colormap creator/editor** (`openColormapCreator`): live horizontal gradient preview with **draggable stop ticks**, slider+number+swatch stop rows, product-assignment chips. **Edit** mode prefills from reconstructed hex stops (`stopsToHexStops`) + assigned products, then delete-recreates and syncs options. **View Stops** shows the real gradient (from `/api/v1/colormap/colors/{name}`) plus the stop table. After any change the frontend calls `POST /api/v1/colormap/cache/invalidate`.
- **Colormap Options** are addable and editable per row (edit = create new pairing + delete old, since there is no PUT).

---

## Indexer
- **COGWatcher:** Polls `ROOT_RADAR_PRODUCTS_PATH` every `SCAN_INTERVAL` seconds (default 30s). First run is a full scan; subsequent runs are incremental (files modified in last 5 min + overlap).
- **COGFilenameParser:** Parses both the current production format (`RADAR_strategy_vol_TIMESTAMP_FIELD.tif`) and the legacy format (`RADAR_TIMESTAMP_FIELD_elev.tif`). Logs a WARNING for legacy files.
- **COGRegistrar:** Extracts COG metadata via rasterio (CRS, bounds, tags), inserts/updates `RadarCOG` records, marks missing files as `MISSING`.
- **`update_radar_activity()`:** Called at the end of every scan cycle. Sets `Radar.is_active = True/False` based on whether a recent AVAILABLE COG exists within the last `RADAR_ACTIVE_THRESHOLD_HOURS` (configurable).
- **TopsAndCoresWatcher:** Scans `TOPS_AND_CORES_DIR` recursively for `*_TOPS_CORES.geojson` files.
- **TopsAndCoresFilenameParser:** Parses `{radar_code}_{strategy}_{vol_nr}_{timestamp}_TOPS_CORES.geojson`. Timestamp format: `YYYYMMDDHHMMSS` (no `T`, no `Z`).
- **TopsAndCoresRegistrar:** Opens GeoJSON, counts cores/tops/features, inserts/updates `TopsAndCores` records.
- `TOPS_AND_CORES_DIR` env var controls the watch root (default `/tops_and_cores`).
- Never modify `COG*` classes when working on tops & cores — they are independent parallel hierarchies.

---

## Frontend
- Leaflet map with radar/product selectors
- Multiple radar selection with opacity control
- Frame animation with speed control (0.5x–2x)
- Periodic polling for new COGs (5 minute interval)
- **Modules:** `v2/app.js`, `v2/map.js`, `v2/animation.js`, `shared/api.js`, `shared/controls.js`, `shared/legend.js`, `shared/tops-cores.js`, `shared/time-wheel.js`
- **One-radar page:** `radar.html` + `v2/radar-app.js` — see dedicated section below.
- **Radar selection order** (`controls.js` → `sortRadarsForDisplay`): active before inactive; within each, RMA group before AR group; numeric ascending with `RMA00` (number 0) sorted last (e.g. `RMA1…RMA17, RMA00, AR5…`).
- **Custom time range** uses a native date input + an iOS-style scroll-snap **TimeWheel** (`shared/time-wheel.js`) for HH:MM. The hidden `#start-date`/`#end-date` `datetime-local` inputs remain the canonical value (read by `getTimeRangeValues`); the date input + wheel only drive them. Call `refreshTimeWheels()` after the panel becomes visible (scroll position can't be set while hidden).
- **Admin panel** is a separate SPA — see the **Admin Panel & Admin API** section above. Linked from the Settings panel.

**Tops & Cores Layer (v2 only):**
- **Module:** `frontend/public/js/shared/tops-cores.js`
- **Class:** `TopsCoresLayer` — manages `L.layerGroup()` of `L.circleMarker` instances
- Cores: `fillColor: '#3b82f6'` (blue). Tops: `fillColor: '#ef4444'` (red). Both: black border, `weight: 1`
- `updateFrame(frame)` fetches `/tops-cores` for ±2.5 min window, then fetches GeoJSON per record via `Promise.all`
- Fire-and-forget from animation loop — never blocks frame advance
- Toggled via settings panel checkbox; point size controlled via 4–20px slider
- State persisted in `localStorage`: `webmet25_tops_cores_visible`, `webmet25_tops_cores_size`

## v2 Frontend Architecture (current production standard)

### File Structure
```
frontend/public/
├── index.html        # Main multi-radar map page
├── radar.html        # One-radar detail page (entry: radar.html?code=AR5)
├── admin.html        # Admin SPA
└── js/
    ├── shared/          # Shared across v1 and v2
    │   ├── api.js       # REST API client
    │   ├── controls.js  # UI control handlers
    │   ├── legend.js    # Legend renderer
    │   ├── tops-cores.js # TopsCoresLayer (L.circleMarker)
    │   └── cog-browser*.js # COG browser alternative view
    └── v2/              # v2-specific (current production)
        ├── app.js       # Main orchestrator — multi-radar map (2300+ lines)
        ├── radar-app.js # One-radar page orchestrator (1500+ lines)
        ├── map.js       # MapManager with L.imageOverlay
        ├── animation.js # AnimationController with requestAnimationFrame
        ├── radar-utils.js # Shared helpers for radar-app.js
        └── constants.js   # Shared constants (MS_PER_HOUR, COVERAGE_MODES, …)
```

### Key differences from v1
| Aspect | v1 | v2 |
|---|---|---|
| Radar layer | L.tileLayer | L.imageOverlay |
| Endpoint | /tiles/{id}/{z}/{x}/{y}.png | /frames/{id}/image.png |
| Animation | setTimeout opacity toggle | requestAnimationFrame |
| DOM objects | ~180 TileLayers per session | 1 overlay per radar |
| HTTP requests | ~1800 per session | ~180 per session |

### One-Radar Page (`radar.html` + `v2/radar-app.js`)

A dedicated single-radar detail view reachable from the main map or directly as
`/radar.html?code=AR5`. Always operates in **CD mode** (vol_nr 01/02, strategy 0315)
with no coverage-mode toggle.

**URL parameters:**
- `code` (required) — radar station code; missing → redirect to `index.html`
- `field` (optional) — initial product key (e.g. `DBZHo`); falls back to `DBZHo` → `COLMAXo` → first available

**Layer system (`state.layers[]`):** Each selected field is a layer object:
```javascript
{
    id, productKey, productTitle,
    opacity,           // 0–1 (default 1.0 for first layer, DEFAULT_FIELD_OPACITY for others)
    visible,           // eye toggle
    colormap,          // object from api.getColormapInfo() — {vmin, vmax, colors, ticks, …}
    selectedColormap,  // overridden colormap name (null = product default)
    vmin, vmax,        // filter bounds (null = no filter; pre-populated from colormap defaults in UI)
    smoothingEnabled,  // Gaussian smooth toggle
    smoothingSigma,    // 0.3–3.0 (default 0.8)
    coverageRadius,    // metres from COG tag (null = full img_radio range)
    zIndex,            // render order
    settingsExpanded,  // collapse state of the Ajustes sub-panel
}
```

**Key functions:**
- `addLayer(productKey)` — fetches colormap, creates layer object, loads frames, calls `renderLayerList()`
- `removeLayer(layerId)` — tears down overlays + frame entries, refreshes display
- `swapLayerField(layerId, newProductKey)` — replaces field in-place; resets vmin/vmax/colormap
- `getTileParamsForLayer(layer)` → `{colormap, vmin, vmax, smooth, smoothSigma}` passed to `_buildFrameUrl`
- `reloadLayerWithNewParams(layer)` — re-fetches all frame images for one layer in parallel;
  does **not** call `renderLayerList()` (strip ticks remain at product defaults, confirming
  the range filter only alpha-masks without changing the colormap normalization range)
- `setLayerColormap(layerId, name)` — fetches new colormap info, re-renders strip, reloads frames
- `showAllLayersAtFrame(index)` — composites all visible layers at a given frame; called on every
  animation tick and on visibility/opacity changes
- `loadLayerFramesForRange(layer, start, end)` — merges new frames into the shared
  `state.mapManager._frameImages` structure; layers share the same timestamp buckets
- `refreshLiveWindow()` — anchors to latest data for first layer, resets frame structure,
  reloads all layers; runs on `LIVE_REFRESH_INTERVAL_MS` timer

**Panel-module-b — Field / Layer selection:**
- Active layer list (`#layer-list`) rendered by `renderLayerList()`: drag-to-reorder handle,
  eye-toggle, field name (click = swap modal), remove button, colormap strip + ticks, opacity
  slider, collapsible "Ajustes" sub-panel with colormap select / range inputs / smoothing
- Collapsible "Añadir campo" section: checkbox list filtered by unfiltered/filtered toggle
  (`state.pickerShowFiltered`); checking a box calls `addLayer`, unchecking calls `removeLayer`
- Swap modal (`#field-picker-modal`): grid of all products; clicking replaces the target layer

**Range filter vs colormap normalization (critical invariant):**
The `vmin`/`vmax` inputs in the Ajustes panel are **filter bounds only** (sent to the frames
endpoint as `?vmin=…&vmax=…`), which the backend uses for **alpha-masking** (pixels outside
the range are transparent). The colormap normalization range always comes from
`colormap_for_field()` (product defaults) — it is never changed by the filter.
The inputs are pre-populated with `layer.colormap.vmin/vmax` when `layer.vmin == null`
so clicking "Aplicar" without narrowing the range sends the full product range (no visual
difference), matching the main page's behavior.

**Coverage rings:** `updateCoverageRadius()` draws one SVG ring per unique
`layer.coverageRadius` value (from the COG tag); the mask cutout uses the largest radius.
`setRadarCoverageRings()` on `MapManager` manages these ring elements.

**Snapshot (`captureMapSnapshot`):** Canvas compositing of basemap tiles + radar overlays +
SVG coverage mask; overlaid with OHMC logo, radar header panel, per-layer colormap strips
(bottom-left, with current timestamp), and a fallback time panel (bottom-right when no
layers are visible).

---

### Animation continuity pattern
Field changes, time-window changes, colormap changes, and range
filter applies NEVER stop the animation. All use staged background
loading via `_loadFramesWithContinuity(loadFn, opts)`:
1. `_fetchTimeRangeFrames()` fetches new frames (pure, no side effects)
2. Animation keeps running from current buffer
3. `animator.setFrames(stagingFrames)` atomically swaps on completion
4. RAF loop picks up new frames on next tick

> ⚠️ Never call `animator.stop()`, `animator.reset()`, or
> `clearRadarLayer()` before new frames are ready. This breaks
> animation continuity. All data loading must go through
> `_loadFramesWithContinuity`.

### Coverage mask (v2 only)
SVG mask inside Leaflet `coverageMaskPane` (zIndex 300).
- Sits above basemap (200) and below radar overlays (400)
- Full-map dark rect with transparent circle cutouts per radar
- `addRadarCoverage(code, lat, lng, radius_m)` — call on radar add
- `removeRadarCoverage(code)` — call on radar remove
- `setCoverageOpacity(opacity)` — called by opacity slider
- Redraws on `viewreset` + `moveend` only (not every frame)
- Zero zoom lag: SVG is child of Leaflet pane transform group

### Basemap
Default: IGN Argenmap (`'argenmap'` key). Persisted to localStorage
(`webmet25_selected_basemap`). Always call `setBasemap(key)` to change —
never manipulate `_baseLayer` directly. Basemaps are defined in the
`BASEMAPS` object in `js/v2/map.js`.

### Frame image rendering
`image-rendering: pixelated` applied to all radar overlays via
`.radar-image-overlay` CSS class. Applied once after `addTo(map)`.
Class persists across `setUrl()` calls — no need to re-apply.

### Legend
Renders: field name (top) → colormap bar → units (bottom).
Unit shows `?` when null. Never mutate `colormap.vmin`/`colormap.vmax`
before calling `legend.render()`. Pass filter range separately:
`legend.render(colormap, { filterVmin, filterVmax })`.

### Timestamps
Frame timestamps displayed in browser OS timezone via
`Intl.DateTimeFormat().resolvedOptions().timeZone`.
UTC with "UTC" suffix is fallback only. No geolocation needed.

### Default values (v2)
| Setting | Default | localStorage key |
|---|---|---|
| Time window | 1.5h (90 min) | timeWindowHours |
| Basemap | IGN Argenmap | webmet25_selected_basemap |
| Coverage opacity | 0.4 | webmet25_coverage_opacity |
| Radar status interval | 300s | radarStatusInterval |
| Live window interval | 60min | liveWindowMinutes |

---

## Tops & Cores Data Pipeline

### Data Flow
radarlib (genpro25-rma*)
→ /tops_and_cores/{radar_code}/YYYY/MM/DD/{radar_code}{strategy}{vol_nr}_{timestamp}_TOPS_CORES.geojson
→ TopsAndCoresWatcher (indexer) scans and registers to DB
→ GET /api/v1/tops-cores — metadata query
→ GET /api/v1/tops-cores/{id}/features — GeoJSON served from disk
→ TopsCoresLayer (v2 frontend) renders L.circleMarker per feature


### GeoJSON Schema (produced by radarlib, consumed by webmet25)
```json
{
  "type": "FeatureCollection",
  "features": [{
    "type": "Feature",
    "geometry": { "type": "Point", "coordinates": [lon, lat] },
    "properties": {
      "type": "core",
      "intensity_dbz": 52,
      "radar_code": "RMA6",
      "observation_time": "2026-05-05T16:38:54Z"
    }
  }]
}
```

- type is "core" or "top"
- Cores carry intensity_dbz (int); tops carry altitude_m (int)
- Coordinates: [lon, lat] — GeoJSON standard order

### Filename Pattern
{radar_code}_{strategy}_{vol_nr}_{timestamp}_TOPS_CORES.geojson
Timestamp format: YYYYMMDDHHMMSS — parsed by TopsAndCoresFilenameParser.
This pattern is the contract between radarlib and webmet25 — never change it
without updating both TopsAndCoresFilenameParser and radarlib's cores_and_tops.py.

### Docker Volumes
Both radar_api and radar_indexer services mount:
```yaml
volumes:
  - ./tops_and_cores:/tops_and_cores:ro
environment:
  TOPS_AND_CORES_DIR: /tops_and_cores
```
The tops_and_cores/ host directory is written by genpro25-rma* containers.
webmet25 services are read-only consumers.

### Migration Notes
- TopsAndCores model uses the existing COGStatus enum — do NOT define a new one
- When writing Alembic migrations that reference cogstatus, use
sqlalchemy.dialects.postgresql.ENUM with create_type=False instead of sa.Enum
(sa.Enum with create_type=False does not reliably suppress creation inside op.create_table)
- manage.py init stamps Alembic head after create_all() — fresh installs
and migration-based upgrades are always in sync

### API Behavior
- GET /tops-cores returns metadata only (not file contents). Cache-Control: no-cache.
- GET /tops-cores/{id}/features reads file from disk. Cache-Control: immutable 86400s.
Updates record status to MISSING in DB if file not found at serve time.
- Empty result from /tops-cores → return [], not 404.

---

## localStorage Keys Reference (v2 Frontend)

| Key | Type | Default | Description |
|---|---|---|---|
| `webmet25_show_inactive_radars` | boolean | false | Show inactive radars toggle |
| `webmet25_show_filtered_fields` | boolean | false | Show filtered fields toggle |
| `webmet25_live_refresh_interval_ms` | number | 300000 | Live refresh interval (ms) |
| `webmet25_coverage_visible` | boolean | false | Coverage circles toggle |
| `webmet25_coverage_opacity` | number | 0.4 | Coverage circles opacity |
| `webmet25_coverage_mode` | string | 'cd' | Active coverage mode id (`'cd'` or `'vig'`). Determines which volumes are queried for products and COGs |
| `webmet25_tops_cores_visible` | boolean | false | Tops & Cores layer toggle |
| `webmet25_tops_cores_size` | number | 8 | Circle marker radius in px |

---

## Coding Conventions & Rules
> Always follow these when generating code.

- **Language version:** Python 3.11
- **Formatter:** black (run before every commit)
- **Linter:** flake8
- **Type hints:** Required on all functions (enforced by mypy)
- **Naming:**
  - Variables and functions: `snake_case`
  - Classes: `PascalCase`
  - Constants: `UPPER_SNAKE_CASE`
- **Error handling:** Always raise specific descriptive exceptions.
  Never use bare `except:` clauses.
- **Database:** Always use transactions for any write operations.
  Never write to the database outside of a transaction block.
- **API:** Never return a 500 error to the client. Always handle 
  missing files and missing records gracefully with proper 4xx 
  responses.

---

## Known Gaps & Technical Debt
> Do not replicate these patterns. Always suggest fixes when 
> touching these areas.

### Critical
- ⚠️ PARTIAL: The public `/api/v1/*` API is still completely open. The admin panel/API (`/admin`, `/api/v1/admin/*`) is gated by **temporary nginx HTTP Basic Auth** (`admin.htpasswd`) — must be replaced with proper JWT before production.
- ❌ No transactions in indexer. Partial failures corrupt DB state.
- ❌ Missing files cause 500 errors. Must be handled gracefully.

### High Priority
- ✅ RESOLVED: L1 LRU + L2 Redis tile cache implemented.
   Frame cache also implemented (key prefix: `frame:`, independent from tile `tile:` prefix).
- ✅ RESOLVED: Gaussian smoothing implemented in frames endpoint (`smooth=true`, `smooth_sigma`).
- ❌ Incomplete error handling in tile rendering and indexer.
- ❌ No rate limiting. API is vulnerable to DOS attacks.
- ❌ Database credentials in plaintext in `docker-compose.yml`.

### Medium Priority
- ✅ RESOLVED: api.js uses relative /api/v1 path unconditionally.
- ✅ RESOLVED: Convective Cores & Storm Tops implemented — radarlib generates GeoJSON,
  indexer registers, API serves, v2 frontend displays as `L.circleMarker` layer via `TopsCoresLayer`.
- ✅ RESOLVED: Coverage Mode Toggle (C+D/VIG) implemented — `COVERAGE_MODES` constant in
  `v2/app.js` defines mode↔volume mapping. Mode persisted to `webmet25_coverage_mode` in localStorage.
- ✅ RESOLVED: Vigilant mode (vol 04) properly disambiguated from C+D (vol 01+02) via `vol_nr`
  column in `radar_cogs` table and `?vol_nr=` filter on `/cogs` endpoint.
- ✅ RESOLVED: Radar activity auto-updated by indexer (`update_radar_activity()`) based on recent COG availability.
- ❌ No pagination on products and references endpoints.
- ❌ No automated tests.
- ❌ No monitoring or log aggregation.

---

## Testing Strategy & Structure
> All tests live in the `tests/` folder.
> Tests run inside the `tests` Docker service defined in 
> `docker-compose.devcontainer.yml`.
> Never add test dependencies to api or indexer requirements.

### Folder Structure
```
tests/
├── api/        # API contract tests using httpx
├── indexer/    # Indexer unit tests
└── e2e/        # Browser tests using Playwright
```

### Running Tests
```bash
# Exec into the tests container
docker exec -it radar_tests bash

# Run all tests
pytest

# Run only API tests
pytest tests/api/ -v

# Run only a specific test file
pytest tests/api/test_health.py -v

# Run E2E tests
pytest tests/e2e/ -v
```

### Test Layers
| Layer | Location | Tool | What it tests |
|-------|----------|------|---------------|
| API Contract | `tests/api/` | pytest + httpx | Every endpoint in the API Contract |
| Indexer | `tests/indexer/` | pytest | File parsing, DB transactions |
| E2E | `tests/e2e/` | pytest + Playwright | Frontend behavior in real browser |

### File Naming
- One test file per router file
- Test file name must match the router it tests
- Examples:
  - `api/app/routers/radars.py` → `tests/api/test_radars.py`
  - `api/app/routers/cogs.py` → `tests/api/test_cogs.py`

### Required Test Pattern
Every test file must follow this exact pattern:
```python
import pytest
import httpx
import os

API_BASE_URL = os.getenv("API_BASE_URL", "http://api:8000")
```

### Required Tests Per Endpoint
Every endpoint must have ALL of the following tests:

1. **HTTP status code test**
   - Happy path must return the correct 2xx status code
   - Example: `test_radars_returns_200`

2. **Response fields test**
   - Response must contain all fields defined in the API Contract
   - Example: `test_radars_response_has_required_fields`

3. **Content type test**
   - Response must return `application/json`
   - Example: `test_radars_returns_json`

4. **Error path test**
   - Invalid inputs must return correct 4xx status codes
   - Never assert a 500 error. A 500 is always a bug, not a 
     valid error response.
   - Example: `test_radar_invalid_code_returns_404`

5. **Data type test**
   - Assert that field types match the contract
     (e.g., strings are strings, numbers are numbers, 
     lists are lists)
   - Example: `test_radars_returns_list`

### Rules
- Always use `API_BASE_URL` from environment variables
- Never hardcode URLs or ports in test files
- Never use pytest fixtures yet, keep tests simple and explicit
- Always test both the happy path AND the error path
- A failing test means the API Contract is violated, 
  not that the test is wrong
- If a test reveals a bug, document it in the Known Gaps section
  of this file before fixing it
- Never skip a test with `@pytest.mark.skip` without adding a 
  comment explaining why

---

## Rules for Writing E2E Tests
> Follow these rules when writing Playwright tests in tests/e2e/
> **How to run them (Docker + bare-machine + CI) is documented in [`docs/E2E_TESTING.md`](../docs/E2E_TESTING.md).** The e2e suite targets the **v2 frontend** (`FRONTEND_URL`, default `http://frontend-v2:80`) and authenticates to `/admin` with `ADMIN_USERNAME`/`ADMIN_PASSWORD` via Playwright `http_credentials` (a shared `tests/e2e/conftest.py` provides the auth context, JS-error capture, and screenshot-on-failure).

### Required Setup
```python
import pytest
from playwright.sync_api import Page

FRONTEND_URL = os.getenv("FRONTEND_URL", "http://frontend:80")
```

### Required Tests Per Frontend Feature
1. **Page loads test:** Assert the page loads without errors
2. **Key element visible test:** Assert critical UI elements are visible
3. **User interaction test:** Simulate real user interactions
4. **API integration test:** Assert the frontend correctly displays 
   data from the API

### Rules
- Always use `FRONTEND_URL` from environment variables
- Test real user flows, not implementation details
- Always take a screenshot on failure for debugging:
```python
page.screenshot(path="tests/e2e/screenshots/failure.png")
```
---

## Rules for Writing Indexer Tests
> Follow these rules when writing tests in tests/indexer/

### Test File Structure
\```
tests/indexer/
├── test_filename_parser.py  # Pure unit tests for COGFilenameParser
├── test_registrar.py        # DB integration tests for COGRegistrar  
└── test_watcher.py          # Scan logic and error resilience tests
\```

### Rules
- test_filename_parser.py must NEVER connect to the database
- test_filename_parser.py must test every filename variation 
  defined in the Output Contract
- Always test both valid AND invalid filenames
- Always test the [o] suffix (raw/non-filtered) separately
- test_registrar.py must verify transaction rollback on failure
- test_watcher.py must verify that one bad file does not stop 
  the entire scan
- Never mock the filename parser in registrar tests, use real 
  filenames from the Output Contract

### Additional Indexer Testing Rules
- Use `@pytest.mark.parametrize` for filename parser tests
---

## 🚀 Spec-Driven Development (SDD) Workflow — STRICTLY ENFORCED
When executing tasks, building features, or fixing bugs from a `.specs/` file, strictly follow this execution loop. Do not skip steps.

### Phase 1: Context & Proposal ⚠️
- Read the specified .specs/ file.
- Read docs/DATA_FLOW.md or docs/COMPONENTS.md depending on the spec's domain.
- DO NOT write implementation code yet.
- Outline your proposed approach in the chat, flag any risks to the radarlib contract, and wait for the user to say "approved".

### Phase 2: Test-First (TDD) 🧪
- Once approved, write the automated tests FIRST based on the Acceptance Criteria in the spec.
- Put tests in the appropriate tests/ subdirectory following the Testing Strategy rules above.
- Do not write the implementation code until the user confirms the tests are ready.

### Phase 3: Implement & Apply 🛠️
- Write the implementation code to make the tests pass.
- Ensure strict compatibility with radarlib's contract.

### Phase 4: Archive & Cleanup 🧹
- After code is applied and tests pass, explicitly ask the user:
  1. "Should I update any documentation (DATA_FLOW.md, COMPONENTS.md, API Contract)?"
  2. "Should I append these changes to the CHANGELOG.md?"
  3. "Please remember to move the spec file to .specs/archived/."

## Known Gaps
- ❌ Pydantic V2 class-based config deprecated in indexer/config.py 
  and radar_db/config.py. Must migrate to ConfigDict before Pydantic V3.

---
> Source: [jgmarti84/gl-webmet25](https://github.com/jgmarti84/gl-webmet25) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-08 -->
