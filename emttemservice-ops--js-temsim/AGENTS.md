## Quick orientation

This repository currently contains a minimal static web project: a single `index.html` at the repo root. There is no build system, package manifest, or server code present in the tree as of this commit.

Use the guidance below to reason about changes and to produce code suggestions that fit the repository's current shape.

## Big picture / architecture (what to assume)

- Single-page static site: `index.html` is the canonical entry point. Treat the project as a static front-end artifact unless new server files appear.
- No frameworks detected (no `package.json`, `webpack`, `vite`, etc.). Code suggestions should avoid adding heavy framework scaffolding unless the user explicitly asks for it.
- Asset and script additions should be colocated next to `index.html` (e.g. `css/`, `js/`) unless the user requests a different structure.

## Developer workflows (explicit commands)

- Run locally (quick): open `index.html` in a browser. For features requiring HTTP (fetch, Service Worker), run a simple static server:

  - Using Node (if Node is available):
    npx http-server . -p 8080

  - Using Python 3 (widely available):
    python -m http.server 8000

  Use the Windows PowerShell prompt; these commands run as-is.

- Debugging: use browser DevTools (Console / Network / Sources). Prefer adding small self-contained `console.log` checks or breakpoint-friendly code rather than complex logging infra.

## Project-specific conventions and patterns (what to follow)

- Keep changes minimal and file-scoped. This repository currently favors small, explicit edits rather than introducing large toolchains.
- If adding JS/CSS, prefer plain files referenced from `index.html` (`<script src="js/app.js"></script>`). If proposing a build system, explicitly state the trade-offs and add a minimal `package.json` and README steps.

## Integration points & external dependencies

- No external services or APIs are present in the repo. If code references external APIs, document the endpoint and any required keys in a new `README.md` and do not hardcode secrets.

## When you propose automation or tests

- Only add test or CI scaffolding when the user asks. If added, include a short README snippet explaining how to run tests locally on Windows PowerShell.

## Examples of useful suggestions (concrete snippets)

- Add a minimal `js/app.js` and include it from `index.html`:

  <!-- index.html -->
  <script src="js/app.js"></script>

- Start a static server in PowerShell using Python:
  python -m http.server 8000

## Merge policy for this file

- If you find existing `.github/copilot-instructions.md` content later, merge preserving any repo-specific items. Prefer the most specific instructions for this repo (entry point: `index.html`).

---
If you want I can (A) open `index.html` and extract/add concrete examples from it into this file, or (B) add a tiny `js/` scaffold and update `index.html` to show the recommended layout. Which would you prefer?

---
> Source: [emttemservice-ops/js-temsim](https://github.com/emttemservice-ops/js-temsim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-18 -->
