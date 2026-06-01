<!--
Short, focused instructions to help an AI coding agent become productive in this repository.
Keep this file concise (20–50 lines). Reference concrete files and commands found in the workspace.
-->

# Copilot instructions for SportPedia

- Repo type: Django web app (Django 5.2.x). Key entry points: `manage.py` and `sportpedia/settings.py`.
- Primary apps: `landingpage` (routes and templates for home/search), `sportlibrary` (models for Sport), `gearguide` (Gear model).

- Quick dev setup:
  - Activate the project's virtualenv: `source env/bin/activate` (venv is in `env/`).
  - Install dependencies if needed: `pip install -r requirements.txt`.
  - Common commands: `python manage.py migrate`, `python manage.py runserver`, `python manage.py createsuperuser`.

- Routing and templates:
  - Root URLs are configured in `sportpedia/urls.py` which includes `landingpage.urls` (note: app folder is `landingpage`; `app_name` is `main`).
  - Templates live in `templates/` (project-level) and app `templates/` folders. Base template: `templates/base.html`. Home page: `landingpage/templates/home.html`.
  - Static assets referenced with `{% load static %}` in templates; static files served via Django staticfiles in dev.

- Database and environments:
  - Development uses SQLite (`db.sqlite3`) by default. Production expects Postgres with env vars (`DB_NAME`, `DB_USER`, etc.) and `PRODUCTION=true` in environment; see `sportpedia/settings.py`.

- Naming & conventions to preserve when editing:
  - Apps: folder `landingpage` but `app_name = "main"` in `landingpage/urls.py`. Use that `app_name` when referencing URL names.
  - Models: `gearguide.models.Gear` uses UUID primary keys. `sportlibrary.models.Sport` exists and is referenced by `gearguide`.
  - Keep templates and static path conventions (project-level `templates/` and `{% static '...' %}` usage).

- Tests & linting:
  - No tests configured in the repo yet. When adding tests, place them in each app's `tests.py` or a `tests/` package.

- Editing guidance for AI agents:
  - Make minimal, focused changes; run `python manage.py migrate` after model changes and `runserver` to smoke-test templates.
  - When adding URLs, update `landingpage/urls.py` and ensure `sportpedia/urls.py` includes the app.
  - Respect existing internationalization/timezone settings in `sportpedia/settings.py`.

- Files to inspect first for context on any UI or data change:
  - `README.md` (project overview and intended features)
  - `templates/base.html`, `templates/search_results.html`, `landingpage/views.py`, `gearguide/models.py`, `sportlibrary/models.py`

- If unclear, ask the maintainer for intended behavior before large refactors (especially changes touching database schema or auth).

Thank you — ask for clarification if an area of the codebase is ambiguous.

---
> Source: [Adamitt/sportpedia](https://github.com/Adamitt/sportpedia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-01 -->
