# Copilot Instructions for PlaceBasedCarbonCalculator.github.io

## Project Overview
- This is a static website for [Carbon & Place](https://www.carbon.place), focused on place-based carbon calculation and visualization.
- The site is organized by topic/feature into subfolders (e.g., `landownership/`, `landuse/`, `pbcc/`, `retrofit/`, `transport/`). Each has its own `index.html`, JS, and settings files.
- Shared assets and logic are in `js/`, `css/`, and `images/` directories. Common JS utilities are in `js/common-nonmap.js`, `js/ui-common.js`, etc.

## Architecture & Patterns
- No build system: All code is plain JS/HTML/CSS, loaded directly by the browser. There is no Node.js, Webpack, or bundler.
- Each feature area (e.g., `landownership`, `pbcc`) is self-contained, with its own data, UI, and settings modules.
- Data is loaded from static files in `data/` or feature subfolders, or from remote APIs (not included in this repo).
- UI logic is modular: look for `ui.js` and `settings.js` in each feature folder for entry points.
- Images and icons are organized by type in `images/` and `assets/`.
- Important libraries are MapLibre, Chart.js, included via CDN or in `js/lib/`.

## Developer Workflows
- No build or test scripts are present; changes are made directly to source files.
- To preview changes, open `index.html` or a feature's `index.html` in a browser. Use a local web server for full functionality (e.g., for service worker support).
- Service worker (`sw.js`) enables offline support; update it if adding/removing files to cache.
- For map-related features, see `map.css`, `manual.js`, and feature-specific JS files.

## Project-Specific Conventions
- Use plain JavaScript (ES5/ES6) and avoid frameworks.
- Keep feature logic isolated in its folder; share only via `js/` utilities.
- Use relative paths for all asset references.
- When adding new features, follow the pattern: create a folder, add `index.html`, `datasets.js`, `settings.js`, `ui.js` as needed.
- For icons/images, add to the appropriate subfolder in `images/` or `assets/`.

## Integration Points
- No external build dependencies; all third-party libraries are included in `js/lib/` or via CDN in HTML.
- Data integration is via static files or remote APIs (API endpoints are not in this repo).

## Key Files & Directories
- `index.html` – main entry point
- `js/` – shared JavaScript utilities
- `css/` – shared and feature-specific styles
- `sw.js` – service worker for offline support
- `pbcc/`, `landownership/`, `landuse/`, `retrofit/`, `transport/` – feature modules
- `images/`, `assets/` – static assets

## Example: Adding a New Feature
1. Create a new folder (e.g., `myfeature/`).
2. Add `index.html`, `datasets.js`, `settings.js`, `ui.js` as needed.
3. Reference shared JS/CSS from `js/` and `css/`.
4. Add assets to `images/` or `assets/`.
5. Update navigation/UI in `index.html` or shared components if needed.

---

For more details, see the folder structure and existing feature modules. This project is intentionally simple and modular for easy extension and maintenance.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PlaceBasedCarbonCalculator)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/PlaceBasedCarbonCalculator)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
