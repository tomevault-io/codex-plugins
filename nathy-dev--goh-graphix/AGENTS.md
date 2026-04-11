<!--
Guidance for AI coding agents working on the GOH-Graphix static site.
Keep this short and focused: reference concrete files, IDs, data attributes and common tasks.
-->

# GOH-Graphix — Copilot instructions

Purpose
- Short, action-first guidance so an AI assistant can be immediately productive on this repo: a small static marketing site (HTML/CSS/vanilla JS) with no backend.

Big picture
- Single-page static website. Entry point: `index.html`. Styling lives in `styles.css`. Behavior is in `script.js`. Images are under `images/`.
- No build system or package manager in the repo. Changes are verified by opening `index.html` in a browser or serving the folder over a simple static server.

Key files to read first
- `index.html` — main structure, navigation, section IDs. Useful examples: `#mobile-menu-toggle`, `#mobile-menu`, `#back-to-top`, `#contact-form`, `#testimonial-carousel`.
- `script.js` — DOM-ready event handlers and component behavior: mobile menu toggle, smooth scrolling, portfolio filtering (uses `data-filter` on `.filter-btn` and `data-category` on `.portfolio-item`), testimonials carousel (uses `.dot` with `data-index`), and a fake contact submit flow (client-side only).
- `styles.css` — CSS variables, responsive breakpoints, and utility classes (e.g., `.container`, `.btn`, `.active`). Prefer reusing existing variables (`--primary`, `--secondary`) when adding styles.
- `images/` — static assets; ensure any new image references are added here and paths in HTML are relative.

Patterns and conventions (project-specific)
- Vanilla JS, no frameworks. Script expects elements to exist by ID/class — avoid changing IDs without updating `script.js`.
- Filtering uses data attributes: filter buttons (`.filter-btn[data-filter="..."]`) and items (`.portfolio-item[data-category="..."]`). Keep those attributes when adding new portfolio items.
- Carousel uses transform on `#testimonial-carousel` and dots with `data-index` — keep the dot count and indexes in sync with testimonial slides.
- Minimal client-side form handling: `#contact-form` has a fake submit that resets and shows an alert. There is no server-side form; integrate a backend or a third-party form service if persistent submissions are required.

Developer workflows
- No build step. Quick local checks:
  - Open `index.html` in a browser.
  - Or run a static server (recommended) to avoid CORS/font issues. Example (PowerShell):
    python -m http.server 8000; # in repo root, then open http://localhost:8000
- Linting/tests: none provided. Keep edits small and verify visually.

Change guidance and examples
- Adding a new portfolio item:
  1. Put the image in `images/`.
  2. Add a `.portfolio-item` block in `index.html` inside `.portfolio-grid` with `data-category` matching existing filters (e.g., `logo`, `flyers`, `branding`).
  3. No JS change needed if category matches existing filter buttons.

- Adding a new filter category:
  1. Add a new filter button: `<button class="filter-btn" data-filter="newcategory">New</button>`.
  2. For items to show under this category, set `data-category="newcategory"` on their `.portfolio-item`.

- Adding a new testimonial slide:
  1. Add a new `.testimonial` inside `#testimonial-carousel`.
  2. Add a matching `.dot` with `data-index` equal to the new zero-based slide index.
  3. The JS reads `dots.length` for total slides; ensure DOM order matches indexes.

Pitfalls to avoid
- Do not rename or remove IDs used by `script.js` (examples: `mobile-menu-toggle`, `mobile-menu`, `contact-form`, `back-to-top`, `testimonial-carousel`, `prevTestimonial`, `nextTestimonial`, `currentYear`).
- Avoid heavy JS rewrites unless converting to a build-based workflow; this repo is intentionally simple and expects vanilla DOM APIs.
- Don't add ES module imports without adding a simple build or changing how files are served.

Integration points
- External fonts and icons are loaded from Google Fonts, RemixIcon CDN, and FontAwesome kit in `index.html` — be mindful of network access when testing locally.

If you edit files
- Keep markup semantic and use existing classes/variables. Reuse CSS variables declared in `styles.css`.
- When adding scripts, append them at the end of `index.html` (before `</body>`) or add to `script.js`.

When to ask the repo owner
- If you need a backend for the contact form, ask whether to integrate a third-party form endpoint (Formspree/Netlify/Custom API) or add a small backend repo.
- If you need automated tests or a build system, clarify preferred tools (Node/npm, Parcel/Vite, or plain python server).

Done — next steps
- If you'd like, I can:
  - Add a brief CONTRIBUTING.md with small dev workflow commands.
  - Wire a simple Python or Node static-server dev task and add a README section.

Please tell me which follow-up you'd like or point out anything missing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Nathy-Dev)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Nathy-Dev)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
