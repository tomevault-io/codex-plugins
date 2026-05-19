<!-- .github/copilot-instructions.md - guidance for AI coding agents in this repo -->
# Copilot instructions for this repo

This is a minimal Flask-based example app. Keep suggestions small, explicit, and focused on the single-file app + templates pattern used here.

- Purpose: Help contributors understand structure, conventions, and safe edits to the simple blog app.

## Big picture
- Single-process Flask app implemented in `blogapp.py` (small, single-file design).
- Templates live in `templates/`: `index.html`, `post.html`, `add_post.html` (present but currently empty).
- Static assets live in `static/` (there is a CSS file named `stle.css` — note the typo).
- The app uses an in-memory list `post` as a temporary database (`post = []` in `blogapp.py`). Any persistence changes should be explicit and limited.

## Key files to inspect first
- `blogapp.py` — main routes, request handling, and how template context is passed (e.g. `render_template("index.html", post=post)`).
- `templates/` — Jinja templates used by routes. Prefer editing templates here rather than embedding HTML in Python.
- `static/stle.css` — CSS file currently present; double-check filename when referencing in templates.

## Patterns & conventions (discoverable in code)
- Keep logic in `blogapp.py` minimal: route handlers should gather data and call `render_template(...)`.
- The project currently uses an in-memory list `post` as its data store — do not assume a database exists.
- Route parameter naming: routes reference `post_id` (example route string found in `blogapp.py`: `/post/<id:int:post_id>`). Preserve parameter names used in handlers.
- Templates are expected to receive variables named in `render_template` calls; follow the exact variable names (e.g., `post`) when updating templates.

## Examples to follow when making changes
- Add a new route that returns a template:

  - In `blogapp.py`: gather data from `post` and call `render_template("post.html", post=item)`.

- When adding static links in templates, reference the existing CSS filename exactly: `/static/stle.css` (or fix the filename consistently across files if renaming).

## Developer workflows (how to run & debug)
- Common quick run (if `app.run()` is present):

  ```powershell
  python blogapp.py
  ```

- If `app.run()` is not present, use Flask CLI from the repo root:

  ```powershell
  $env:FLASK_APP = 'blogapp.py'; flask run
  ```

- Use the browser to inspect templates at `/` and `/post/<id>`.

## What to avoid / gotchas
- Do not replace the in-memory `post` list with a new persistence layer without documenting the migration and test steps.
- Be careful with route converter syntax — maintain parameter names expected by handler functions.
- Templates and static files are currently empty; create minimal, well-formed Jinja templates and link CSS by the exact static path.

## If you change structure
- Update this file with any new files, directories, or developer commands discovered.

## Questions for the author
- Should `static/stle.css` be renamed to `static/style.css`? If yes, update templates and mention the rename here.
- Do you expect persistent storage? If so, mention preferred DBs or ORMs so agents can propose consistent changes.

If anything here is unclear or you want more automation (tests, run scripts, CI), tell me what to include and I will iterate.

---
> Source: [Oghenerukevwe5/simple-blog-app](https://github.com/Oghenerukevwe5/simple-blog-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-19 -->
