# CLAUDE.md - Project Instructions

## Project Overview

Static portfolio blog for a Cloud & Platform Engineer. Showcases projects across Cloud Engineering, SRE, DevOps, Platform Engineering, AI, and MLOps.

## Tech Stack

- **Plain HTML/CSS/JS** — no build tools or frameworks
- **Fonts**: DM Sans (body), JetBrains Mono (code/technical)
- **Theme**: Dark mode, "Technical Precision" aesthetic (see `DESIGN.md`)
- **Data**: Projects stored in `data/projects.json`, loaded via `fetch()`

## Project Structure

```
index.html          — Home page (hero + featured projects + blog preview)
projects.html       — Filterable project grid
project.html        — Single project detail (uses ?slug= query param)
blog.html           — Blog listing
timeline.html       — Chronological activity feed
about.html          — Background, skills matrix
css/styles.css      — All styles (CSS custom properties in :root)
js/main.js          — All JS (project loading, filtering, nav)
data/projects.json  — Project data (source of truth)
content/projects/   — YAML templates for project content
projects-schema.json — JSON Schema for project entries
DESIGN.md           — Full design spec and implementation phases
```

## Key Conventions

- **No build step** — files are served directly as static HTML
- **Single CSS file** — all styles in `css/styles.css` using CSS custom properties
- **Single JS file** — all logic in `js/main.js`; page-specific behavior is gated by checking for DOM element IDs
- **Project data** — add/edit projects in `data/projects.json` following the schema in `projects-schema.json`
- **HTML escaping** — use the `escapeHtml()` function in `main.js` when rendering user-facing content
- **Categories** — must be one of: Cloud Engineering, SRE, DevOps, Platform Engineering, AI, MLOps

## Design Tokens (CSS Custom Properties)

- `--bg-primary: #0d1117` / `--bg-secondary: #161b22` / `--bg-tertiary: #21262d`
- `--text-primary: #e6edf3` / `--text-secondary: #8b949e`
- `--accent: #58a6ff` / `--accent-hover: #79b8ff`
- `--border: #30363d`

## Adding a New Project

1. Add an entry to `data/projects.json` matching the schema in `projects-schema.json`
2. Required fields: `title`, `slug`, `date`, `categories`, `description`
3. Optional: `technologies`, `status`, `features`, `links`
4. Optionally add a YAML template in `content/projects/` for reference

## Notes

- No package manager or dependencies — just open HTML files in a browser or serve with any static server
- Not currently a git repository

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/4LPH4-V1P3R) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
