# Radar Visualization Tool - AI Agent Guide

## Project Overview
Web-based interactive meteorological radar visualization tool processing NetCDF radar files. Developed as a university thesis project (FAMAF UNC). **Backend** (FastAPI) processes radar data; **Frontend** (React + Leaflet) renders interactive maps with tiled overlays.

## Architecture

### Core Data Flow
1. **Upload** → NetCDF files stored in `backend/app/storage/uploads/`
2. **Process** → PyART grids radar data → Rasterio generates GeoTIFF → COG (Cloud Optimized GeoTIFF) with overviews
3. **Serve** → TiTiler dynamically tiles COGs → Frontend renders via Leaflet using TileJSON endpoints
4. **Cache** → In-memory LRU cache (`backend/app/core/cache.py`) stores 2D/3D grids to avoid re-gridding (200-600 MB limits)

### Backend Structure (`backend/`)
- **`app/main.py`**: FastAPI app with TiTiler COG factory at `/cog` prefix
- **`app/routers/`**: API endpoints (`process.py`, `upload.py`, `pseudo_rhi.py`, `radar_stats.py`, `radar_pixel.py`)
- **`app/services/radar_processor.py`**: Core processing logic - PyART gridding, CAPPI/PPI/COLMAX/RHI products, COG generation
- **`app/services/radar_common.py`**: Shared utilities (field resolution, colormaps, gate filters, QC masking)
- **`app/utils/`**: PNG generation, CAPPI helpers, colormap definitions
- **`app/core/`**: Config (`settings`), cache (LRU), constants (field aliases, render params)
- **`app/schemas.py`**: Pydantic models for all API contracts

### Frontend Structure (`frontend/src/`)
- **`App.jsx`**: Main orchestrator - state management, API calls, multi-radar frame merging by timestamp
- **`components/MapView.jsx`**: Leaflet map with `COGTile` component fetching TileJSON from TiTiler
- **`api/backend.js`**: Axios client for all backend endpoints
- **`components/ProductSelectorDialog.jsx`**: UI for product/field/filter selection
- **`components/PseudoRHIDialog.jsx`**: Vertical transect generation UI with point picking
- **`components/AreaStatsDialog.jsx`**: Polygon-based stats extraction

## Critical Conventions

### 1. File Naming Convention
NetCDF files **must** follow: `{RADAR}_{STRATEGY}_{VOLUME}_{TIMESTAMP}Z.nc`
- Example: `RMA1_0315_01_20250819T001715Z.nc`
- Parsed by `backend/app/utils/helpers.py::extract_metadata_from_filename()`
- Used for radar identification, volume filtering, timestamp-based animation grouping

### 2. Field Resolution System
Fields are resolved through aliases defined in `backend/app/core/constants.py::FIELD_ALIASES`:
```python
FIELD_ALIASES = {
    "DBZH": ["DBZH", "corrected_reflectivity_horizontal"],
    "RHOHV": ["RHOHV", "rhohv"],
    # ...
}
```
- Use `radar_common.py::resolve_field()` to find actual field name in radar object
- Each field has render defaults in `FIELD_RENDER` (vmin/vmax/colormap)

### 3. Cache Strategy
**Two-tier caching** (`backend/app/core/cache.py`):
- `GRID2D_CACHE`: Stores collapsed 2D grids (post-PyART interpolation, pre-colormap) - 200 MB
- `GRID3D_CACHE`: Stores full 3D grids for vertical transects - 600 MB
- Cache keys include file hash, product, field, elevation/height, volume, but **NOT filters** (filters applied post-cache as masks)
- QC fields (RHOHV, etc.) cached separately alongside main field for post-grid masking

### 4. Filter Application
**Post-grid masking** approach (changed from older pre-grid gatefiltering):
- Filters defined as `RangeFilter` in `schemas.py` (field, min, max)
- QC filters (fields in `AFFECTS_INTERP_FIELDS` like RHOHV) applied as 2D masks after gridding
- Visual filters (same field as main) applied as direct masks on cached 2D array
- See `radar_processor.py::process_radar_to_cog()` lines ~420-470 for implementation

### 5. Multi-Radar Frame Merging
Frontend (`App.jsx::mergeRadarFrames()`):
- Groups layers from different radars by timestamp with 240-second tolerance
- Backend returns `ProcessResponse.results[]` (per-radar) → Frontend merges into unified frames
- Each frame contains multiple `LayerResult` objects sorted by `.order` field

### 6. COG Generation Pipeline
In `radar_processor.py::process_radar_to_cog()`:
1. Check cache for existing COG (early return if exists)
2. PyART reads NetCDF → builds 3D grid with `grid_from_radars()`
3. Collapse 3D → 2D via `collapse_grid_to_2d()` (PPI follows beam, CAPPI takes Z-slice, COLMAX takes max)
4. PyART's `write_grid_geotiff()` writes RGB GeoTIFF warped to Web Mercator
5. `convert_to_cog()` uses Rasterio COG driver with overviews (factors: 2,4,8,16) + DEFLATE compression
6. Return `LayerResult` with `tilejson_url` pointing to TiTiler endpoint

## Development Workflows

### Backend Setup (Windows)
```bash
conda create -n radar-env -c conda-forge python=3.11 gdal rasterio pyproj shapely
conda activate radar-env
pip install -r backend/requirements.txt
cd backend
make run  # or: uvicorn app.main:app --reload --port 8000
```

**Linux alternative**: `python3 -m venv venv && source venv/bin/activate && sudo apt install gdal-bin libgdal-dev libhdf5-dev libnetcdf-dev`

### Frontend Setup
```bash
cd frontend
npm install
npm run dev  # Vite dev server on localhost:3000
```

### Key Backend Endpoints
- `POST /upload`: Multipart file upload → validates extensions/size → returns metadata + volumes/radars
- `POST /process`: Main processing → accepts `ProcessRequest` (files, product, fields, elevation, height, filters, selectedVolumes, selectedRadars)
- `POST /process/pseudo_rhi`: Vertical cross-section between two lat/lon points
- `POST /stats/area`: Compute statistics over GeoJSON polygon
- `POST /stats/pixel`: Single-pixel value extraction
- `GET /cog/WebMercatorQuad/tilejson.json?url=<path>`: TiTiler dynamic tiling endpoint

### Testing Changes
- Backend: Modify code → FastAPI auto-reloads (uvicorn `--reload`)
- Frontend: Vite HMR auto-updates browser
- Check browser console for TileJSON fetch errors (COGTile component)
- Backend logs show PyART processing errors, cache hits/misses

## Project-Specific Gotchas

### GDAL Installation
- **Windows**: Pre-built wheels in `backend/` (GDAL-3.9.2, gdal-3.11.1)
- Conda is strongly preferred on Windows (`conda install gdal` via conda-forge)
- Linux: system packages required before pip install

### PyART Custom Fork
`requirements.txt` installs from custom fork:
```
arm_pyart @ git+https://github.com/IgnaCat/pyart.git@84f411ae86e05b14fc075b8f6535af84c8bba2c9
```
Likely contains project-specific colormaps or fixes.

### TiTiler URL Construction
- COG paths must be **absolute POSIX paths** (`Path.resolve().as_posix()`)
- Query params: `resampling=nearest&warp_resampling=nearest` for crisp radar pixels
- Frontend adds cache-buster timestamp to tile URLs to prevent cross-product contamination

### Static File Serving
`main.py` mounts `app/storage/tmp` at `/static/tmp` for direct COG access. TiTiler uses file URIs, not HTTP URLs.

### Volume-Based Processing
- Volume `03` has lower resolution (300m grid) vs others (1000m) - see `radar_processor.py` line ~343
- Volume `03` invalid for PPI product (warning generated by `process.py`)

### Memory Management
- Parallel processing uses `ThreadPoolExecutor` with `max_workers = min(max(4, cpu_count*2), items*fields)` 
- LRU cache auto-evicts based on byte size (not item count)
- Temporary GeoTIFFs cleaned after COG conversion

## Key Files to Understand First
1. `backend/app/services/radar_processor.py` - Core processing logic
2. `backend/app/routers/process.py` - Multi-radar orchestration
3. `frontend/src/App.jsx` - State management and frame merging
4. `backend/app/core/constants.py` - Field mappings and render params
5. `backend/app/schemas.py` - API contracts

## Common Tasks

### Adding a New Radar Product
1. Add to `ALLOWED_PRODUCTS` in `backend/app/core/config.py`
2. Implement collapse logic in `radar_processor.py::collapse_grid_to_2d()`
3. Update frontend `ProductSelectorDialog.jsx` options

### Adding a New Field
1. Add alias to `FIELD_ALIASES` in `constants.py`
2. Add render params to `FIELD_RENDER`
3. Add unit to `VARIABLE_UNITS`
4. No frontend changes needed (dynamic population)

### Debugging Cache Misses
- Check `grid2d_cache_key()` in `radar_common.py` - ensures consistent key generation
- Cache keys include file hash, product, field, elevation/height, volume, interpolation method
- Filters **not** in cache key (intentional - applied post-cache)

### Fixing TileJSON 404s
- Verify COG file exists at path in `tilejson_url`
- Check TiTiler logs for GDAL errors (usually CRS issues)
- Ensure `warp_to_mercator=True` in `write_grid_geotiff()` call

---
> Source: [IgnaCat/radar-visualization-tool](https://github.com/IgnaCat/radar-visualization-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-13 -->
