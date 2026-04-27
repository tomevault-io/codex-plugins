# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Alberi Monumentali Italia** — A QGIS plugin (Python/PyQt5-PyQt6) that downloads Italy's monumental trees registry from the MASAF (Ministero dell'agricoltura, della sovranità alimentare e delle foreste) website, converts coordinates from DMS to WGS84 decimal degrees, and saves the data as Shapefile or GeoPackage with graduated symbology.

- QGIS minimum version: 3.20
- All UI text and messages are in Italian
- Data source: [MASAF official page](https://www.masaf.gov.it/flex/cm/pages/ServeBLOB.php/L/IT/IDPagina/11260) — CC BY 4.0

## Installation / Deployment

There is no build system. To install the plugin in QGIS:
1. Package all files into `alberi_monumentali.zip`
2. QGIS → Plugin → Gestisci e installa plugin → Installa da ZIP

Dependencies: `xlrd` (bundled with QGIS) and `pandas`. No `requirements.txt` or package manager.

## Architecture

The plugin uses a clean 5-module design:

- **`__init__.py`** — QGIS entry point, exposes `classFactory()`
- **`main.py`** — `AlberiMonumentali` class: plugin lifecycle (menu, toolbar, `initGui`, `unload`, `run`)
- **`dialogs.py`** — `AlberiDialog` (PyQt5/6 UI): region multi-select, format chooser, progress bar, status messages; spawns `DownloadWorker` threads and connects Qt signals for async updates
- **`downloader.py`** — `DownloadWorker(QThread)`: scrapes live URLs from MASAF via regex, falls back to hardcoded URLs for all 21 regions; exposes `progress`, `region_done`, `finished`, `error` signals; supports abort
- **`converter.py`** — Pure conversion pipeline: `dms_to_dd()` parses Italian DMS strings (comma decimals, Unicode degree symbols), `read_xls()` loads Excel via pandas+xlrd, `dataframes_to_layer()` builds a `QgsVectorLayer` in memory, validates coordinates against Italy's bbox (lat 36–48, lon 6–19), and writes SHP or GPKG to disk
- **`symbology.py`** — `apply_graduated_symbology()`: 5-class graduated renderer by trunk circumference (`CIRCF_CM`/`CIRCONFERENZA_FUSTO_CM`), color range `#A8D5A2`→`#0D3311`; `_apply_labels()` shows species name above 1:50,000 zoom

## Key Technical Details

**Field names differ by format** — Shapefile enforces a 10-character limit; two dictionaries in `converter.py` (`FIELD_MAP_SHP`, `FIELD_MAP_GPKG`) handle the mapping. The calculated fields `LAT_DD`/`LON_DD` (decimal degrees) are always added regardless of format.

**Column detection is fuzzy** — `find_columns()` in `converter.py` normalizes column names (lowercase, alphanumeric-only) to handle varying MASAF update schemas.

**URL scraping with fallback** — On dialog open, `scrape_regional_urls()` fetches the MASAF page and extracts current `.xls` links via regex. If scraping fails, hardcoded URLs (last updated 23/10/2025, 9th national update) are used.

**CRS** — WGS84 (EPSG:4326) is hardcoded throughout; no reprojection is performed.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pigreco) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-11 -->
