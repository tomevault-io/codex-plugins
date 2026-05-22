## Kytchen Table — Copilot instructions

This file contains concise, actionable guidance to help AI coding agents be immediately productive in the Kytchen Table repository (Django project).

- Project type: Django 4.2 web app (see `requirements.txt`). Apps live in top-level folders: `home`, `menu`, `reserve`, `my_account`, `philosophy`.
- Entry points:
  - Main settings: `kytchen_table/settings.py` — environment-driven (expects `SECRET_KEY`, `DATABASE_URL`).
  - URL routing: `kytchen_table/urls.py` — maps top-level pages (`home`, `menu`, `philosophy`, `reserve`, `my_reservations`).
  - WSGI entry: `kytchen_table/wsgi.py`; Procfile uses `gunicorn kytchen_table.wsgi` for production.

- Key patterns & conventions (concrete examples):
  - Views are predominantly function-based in each app's `views.py` (e.g. `home.views.home`, `reserve.views.reservation`).
  - Templates live in `templates/` and per-app subfolders (e.g. `templates/home/index.html`, `templates/menu/menu.html`).
  - Static assets live under `static/` and served in dev via Django; `whitenoise` is enabled in middleware for simple static serving in production.
  - Authentication uses `django-allauth` (routes under `accounts/` in `kytchen_table/urls.py`).
  - Cloudinary is used for media storage (`cloudinary`, `cloudinary_storage` in `INSTALLED_APPS`).
  - Database is configured via `dj_database_url.parse(os.environ.get('DATABASE_URL'))` in `settings.py` — prefer setting `DATABASE_URL` env var or local `env.py` file.

- How to run locally (discovered from project files):
  1. Create a venv and install deps: `python -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt`
  2. Provide local secrets (either create an `env.py` at repo root or export env vars):
     - `SECRET_KEY`, `DATABASE_URL` (for local dev you can use SQLite by setting `DATABASE_URL=sqlite:///db.sqlite3`).
  3. Run migrations and create superuser: `python manage.py migrate` then `python manage.py createsuperuser`.
  4. Run dev server: `python manage.py runserver` (or use Procfile + Heroku-style `gunicorn` for prod).

- Testing & checks:
  - Tests are available per-app in `tests.py`. Run: `python manage.py test`.
  - Typical dev debug: `python manage.py shell` and `python manage.py showmigrations`.

- Integrations & external dependencies:
  - `django-allauth` for authentication (`accounts/` URL include).
  - `cloudinary` for image/media storage (ensure Cloudinary env vars for upload in production).
  - `dj_database_url` for parsing `DATABASE_URL` (12-factor friendly).
  - `gunicorn` used in `Procfile` for production WSGI server.

- Project-specific quirks & things to watch for (concrete notes):
  - `DEBUG = True` in `settings.py`; treat this as dev-only — production should set `DEBUG=False` and provide `SECRET_KEY` via env.
  - `DATABASES` is set to parse `DATABASE_URL`; default SQLite config is commented out — local setups can either create `env.py` or set `DATABASE_URL` to `sqlite:///db.sqlite3`.
  - Templates rely on `django.template.context_processors.request` (request is available in templates).
  - Some app URL files may contain inconsistencies (example: `menu/urls.py` references a `philosophy` view). If making routing changes, confirm which top-level `urlpatterns` are authoritative (see `kytchen_table/urls.py`).
  - All apps expose views in `views.py` and often rely on small helper forms in `forms.py` (e.g., `reserve/forms.py`) — prefer editing forms and templates together.

- When editing or adding models:
  - Follow existing pattern: define model in `models.py`, add migration with `python manage.py makemigrations <app>`, and apply with `migrate`.
  - Example: `menu.models` uses ManyToMany for allergens (see README discussion) — keep allergen choices logic in the model and avoid duplicating choices list in forms.

- Good quick examples to reference in the repo:
  - Routing examples: `kytchen_table/urls.py` (top-level mapping)
  - Settings and env patterns: `kytchen_table/settings.py`
  - Procfile and production entry: `Procfile`
  - Dependency pins: `requirements.txt`

- If you need clarification from humans:
  - Confirm intended production DB and Cloudinary credentials before touching media/storage code.
  - Ask which branches or deploy environment (Heroku vs local) are targeted for changes.

If any part of this file is unclear or you'd like more examples (for instance a short checklist for making a trivial feature change), tell me which area to expand and I'll iterate.

---
> Source: [foster95/kytchentable](https://github.com/foster95/kytchentable) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-22 -->
