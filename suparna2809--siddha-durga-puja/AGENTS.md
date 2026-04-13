# Copilot Instructions for Siddha Durga Puja Website

This project is a static website for the Siddha Durga Puja committee, built with HTML, Tailwind CSS, and vanilla JavaScript. Use these guidelines to maximize productivity and maintain consistency when contributing with AI coding agents.

## Project Structure & Key Files
- All main pages are in the root directory as `.html` files (e.g., `index.html`, `about.html`, `gallery.html`).
- All static assets are under `assets/`:
  - `assets/css/styles.css`: Custom styles (overrides Tailwind where needed).
  - `assets/js/main.js`: Shared JS for navigation, sliders, etc.
  - `assets/js/nirghantan.js`: Glossary logic.
  - `assets/js/gallery.js`: Gallery/lightbox logic.
  - `assets/news.json`: News data for dynamic sections.
  - `assets/images/`: All images, icons, and PDFs.

## Patterns & Conventions
- **No build step:** All files are static. Do not add Node.js, Python, or other build tools.
- **Styling:** Use Tailwind CSS utility classes in HTML. Use `assets/css/styles.css` for customizations or overrides.
- **JavaScript:** Keep JS modular and page-specific. Shared logic goes in `main.js`. Avoid frameworks.
- **Data:** News and glossary data are in JSON or JS arrays, not fetched from a backend.
- **Navigation:** Navigation is duplicated in each HTML file. Update all relevant files for nav changes.
- **Images:** Reference images with relative paths from `assets/images/`.
- **No server-side code:** Do not add PHP, Node.js, or backend logic.

## Developer Workflow
- To preview changes, open `index.html` (or any page) directly in a browser, or use a live server extension.
- No tests or CI/CD are present or required.
- For new pages, copy an existing HTML file as a template.
- For new JS/CSS, place files in the appropriate `assets/` subfolder and link them in the HTML.

## Examples
- To add a new event to the news section, update `assets/news.json` and the relevant HTML.
- To add a new gallery image, place it in `assets/images/` and update `gallery.html`.
- To change the color scheme, edit variables in `assets/css/styles.css`.

## Do Not
- Do not introduce frameworks, build tools, or server-side code.
- Do not use absolute URLs for local assets.
- Do not change the static nature of the site.

Refer to `README.md` for further details on file purposes and customization.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/suparna2809)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/suparna2809)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
