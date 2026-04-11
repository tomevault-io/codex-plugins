# Copilot Instructions for Morningate Game Studio Website

## Project Overview
- This is a creative sandbox for the Love & Sharing AI project, blending static web content, interactive storybooks, and a local admin API for media management.
- The site is built for local development in VS Code and static hosting (GitHub Pages, Netlify).

## Architecture & Key Components
- **Static Site:** Main UI in `index.html`, with supporting pages in `pages/` and storybooks in `stories/wordless/*`.
- **Admin Panel:** Modal UI in `index.html` and logic in `scripts/admin_panel.js` for uploading and managing storybook pages.
- **Backend API:** `admin_server.py` (Flask) provides `/api/storybooks` and `/api/upload` endpoints for admin panel integration. See `requirements.txt` for dependencies.
- **Storybooks:** Each wordless book is a folder under `stories/wordless/`, following the template in `template_storybook/`. Each has its own `index.html`, scripts, and styles.
- **Harbinger Novel (legacy content):** Chapters live in `stories/harbinger/chapters/` as markdown files. The standalone Harbinger viewer has been removed from the site for now.

## Developer Workflows
- **Run Local Admin API:**
  ```powershell
  python -m pip install -r requirements.txt
  python admin_server.py
  ```
- **Static Site Preview:**
  Open `index.html` or use VS Code Live Server.
- **Add New Storybook:**
  Duplicate `stories/wordless/template_storybook/` and update as needed.
- **Upload Pages:**
  Use the admin panel (homepage button) to drag-and-drop images; server normalizes and stores them.

## Conventions & Patterns
- **Naming:** Storybook folders and chapter files use descriptive, lower_snake_case names. Update all references when renaming.
- **Markdown Links:** Always update links in markdown and HTML when moving/renaming files.
- **Image Normalization:** All uploads are resized to 1024px height by default (see `admin_server.py`).
- **Security:** Set `LS_ADMIN_TOKEN` for protected uploads; client sends `X-Admin-Token` header.
- **Minimal Dependencies:** Only Flask and Pillow required for backend; frontend is vanilla JS/CSS/HTML.

## Integration Points
- **Frontend ↔ Backend:** Admin panel JS calls Flask API at `http://localhost:5001`.
- **Storybook Display:** Readers in `stories/wordless/*/index.html` use local scripts for interactivity.
 

## Examples
- To rename a storybook folder, update all references in scripts, markdown, and HTML (see recent `harbinger_dawn → dawn_remembered` change).
- To add a new chapter, place a markdown file in `stories/harbinger/chapters/` and update any viewers or links.

## References
- See `README.md` for setup and local API details.
- See `stories/wordless/template_storybook/README.md` for storybook conventions.
- See `admin_server.py` for backend API details.

---
For questions or unclear patterns, review the latest commit messages or ask for clarification in project documentation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Contingencyplana)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Contingencyplana)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
