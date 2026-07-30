# Copilot instructions

## Build, test, and lint
- No build, test, or lint tooling is configured in this repo (static HTML/CSS/JS).

## High-level architecture
- Static multi-page GitHub Pages site. Entry points are the HTML pages in the repo root (`index.html`, `status.html`, `tasks.html`, `team.html`, `future.html`).
- Shared presentation lives in `style.css` and the animated background in `matrix.js`, both linked by every page.
- `matrix.js` injects a canvas into the `#matrix-bg` container and drives the particle animation; every page includes `<div class="matrix-bg" id="matrix-bg"></div>` and loads `matrix.js` at the end of `<body>`.
- `tasks.html` is the only page with app-like behavior: it loads Chart.js from a CDN and fetches currency data from the NBU API (with a CORS proxy) to populate tables and charts.

## Key conventions
- Navigation markup is duplicated across pages; the active page is marked by `class="active"` on the `<li>` element (not the `<a>`).
- UI layout relies on shared utility classes (`container`, `section`, `card-grid`, `terminal`, `code-block`) that are styled in `style.css`. Reuse these instead of inventing new layout patterns.
- Formatting is governed by `.editorconfig`: 2-space indentation, LF line endings, UTF-8, and final newlines for non-Markdown files.

---
> Source: [andrewmatsur/lab8](https://github.com/andrewmatsur/lab8) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-30 -->
