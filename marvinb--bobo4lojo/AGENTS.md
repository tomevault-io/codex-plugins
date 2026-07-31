<!-- GitHub Copilot / AI agent guidance for the Bobo4Lojo repo -->

# Quick Context

This repository is a small static campaign website. The primary entry is `index.html` (root). The site is served as static files (there's a `_config.yml` referencing a Jekyll theme and a `CNAME`, so the repo is published via GitHub Pages), but the actual pages are standalone HTML using the Tailwind CDN.

# What to know before editing

- **Entry point:** `index.html` — contains the full page markup and Tailwind utility classes.
- **Static assets:** `assets/` — images and media; `style.css` and `script.js` are linked from the root.
- **Jekyll note:** `_config.yml` sets `theme: jekyll-theme-Modernist`. There is not a full Jekyll site structure in the repo, so treat this primarily as a static site unless a full Jekyll build is intentionally added.
- **Hosting:** `CNAME` present — indicates GitHub Pages is used for publishing.

# Codebase patterns & repository-specific gotchas

- Tailwind is used via CDN in `index.html` (no Node/Tailwind build). Changes to layout are typically done by editing Tailwind classes in HTML rather than running a CSS build.
- `style.css` exists alongside Tailwind and contains global rules (e.g., `body { background-color: #eb0808; }`). Expect potential style conflicts: decide whether to prefer Tailwind utilities or `style.css` overrides.
- `script.js` contains a small DOM-ready handler that targets `#main-title`. However, `index.html` does not currently include an element with `id="main-title"` — `h1` in the header has no id. When adding behavior, either add `id="main-title"` to the intended element or update `script.js` to select the correct node.
- Prefer relative asset paths (e.g., `assets/096EE62E-DB4D-4CE0-807B-95B6A55B267E.jpeg`) and keep asset filenames unchanged unless intentionally replacing.

# Editing and previewing locally

- Quick static preview (no dependencies):

  ```bash
  # start a simple file server from repo root
  python3 -m http.server 8000
  # then open http://localhost:8000
  ```

- If you want to use Jekyll (optional):

  ```bash
  # only valid if you add a Gemfile and install bundle; otherwise skip
  bundle exec jekyll serve --livereload
  ```

# Useful grep/search patterns (examples)

- Find JS/CSS id mismatch: `grep -Rn "main-title" .`
- Find Tailwind usage: `grep -Rn "cdn.tailwindcss.com" .` or search for utility classes in `index.html`.

# Commits & PR guidance for AI edits

- Make minimal, focused commits. Example commit message for AI-generated changes: `chore: add copilot guidance` or `fix: align script id with DOM`.
- If modifying `style.css`, note that Tailwind utilities may be preferred for layout changes.

# Where to look for follow-up

- `index.html` — UX, structure, and classes.
- `script.js` — tiny client behavior (update selectors if you change markup).
- `style.css` — global overrides that may conflict with Tailwind.
- `_config.yml` and `CNAME` — publishing hints (GitHub Pages / Jekyll).

# If something is unclear

Ask for clarification or examples of desired behavior (design changes, JS interactions, or how to reconcile Tailwind vs custom CSS). If you want, I can open a PR that (a) adds `id="main-title"` to the header `h1` and updates `style.css`/`script.js` accordingly, or (b) convert global styles to Tailwind utilities.

---
> Source: [marvinb/Bobo4Lojo](https://github.com/marvinb/Bobo4Lojo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-31 -->
