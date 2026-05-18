## Quick orientation for automated coding agents

This repository is a small Django-based video streaming admin/viewer app. The guidance below highlights project-specific conventions, important files, and common dev tasks so an AI agent can be productive immediately.

High-level architecture
- Project root contains `manage.py` and the `core` package. The main application lives at `core/StreamingApp`.
- Models and business logic: `core/StreamingApp/models.py` — most entities use UUID primary keys (e.g. `series_id`, `episode_id`, `user_id`). Expect ManyToMany through models (e.g. `SeriesGenre`) and case-insensitive unique constraints (see `Producer`/`Studio`).
- Views: `core/StreamingApp/views.py` — mix of class-based views (admin and viewer) and a few function views (notably `save_progress` and the delete spotlight helper). Class-based views are exported at the bottom as aliases (e.g. `play_episode = PlayEpisodeView.as_view()`).
- Templates: `core/templates/` organizes templates by role (admin/guest/viewer/base_user/base_admin). Templates often assume context variable names like `series`, `featured_series`, `episode`, `origin`.
- Static & media: media files live under `media/` and uploaded images use ImageField paths like `series_thumbnails/`, `profile_pictures/`, `spotlight_series_images/`.

Key project-specific patterns
- UUID primary keys everywhere — views/URLs expect UUID strings. Use `series.series_id` / `episode.episode_id` when constructing URLs or lookups.
- Role-based guard patterns:
  - Admin pages use `AdminRequiredView` (redirects non-admins to `homepage`).
  - Viewer login uses `ViewerRequiredView` which intentionally logs out any authenticated user to force a fresh viewer session.
  - Many view methods check `request.user.role` explicitly.
- Forms: `core/StreamingApp/form.py` contains forms used in admin flows (e.g. `AddSeriesForm`, `AddEpisodeForm`). Admin create/update flows always pass `request.FILES` when handling image uploads.
- YouTube embedding: Episodes store a `video_id` and templates build an iframe with `https://www.youtube.com/embed/{{ episode.video_id }}?...&origin={{ origin }}`. The code computes `origin = f"{request.scheme}://{request.get_host()}"` in views before rendering.
- AJAX save_progress endpoint: `save_progress(request)` is a POST JSON endpoint used from the front-end (guest watch page) to persist watch progress and increment `Episode.view_count` on initial progress > 0. It returns JSON statuses (`ok`, `forbidden`, `bad_request`, `not_found`).

Run, build and debug hints
- Environment: `core/settings.py` uses `environ` and loads `.env` in the project root. Important vars: `SECRET_KEY`, `DEBUG`, `ALLOWED_HOSTS`.
- Local dev server: use the normal Django commands. From repo root:

```
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt  # add if not present; project may rely on pillow, django-environ
python manage.py migrate
python manage.py createsuperuser  # to access admin flows
python manage.py runserver
```

- Static/media served in development using Django's settings: MEDIA_URL = '/media/' and MEDIA_ROOT = <repo>/media. Templates rely on `{{ series.thumbnail_picture.url }}` and `{{ user.profile_picture_url.url }}`. Ensure uploaded test images are placed under `media/` or use the admin upload UI.
- Tests: there are no automated tests included in the repo root. If adding tests, follow existing app structure under `core/StreamingApp/tests.py` and use Django's TestCase.

Code navigation & hotspots for changes
- When changing view routing or adding endpoints, update `core/StreamingApp/urls.py` and the view alias at the bottom of `views.py`.
- When modifying models with new fields or constraints, add a migration (`python manage.py makemigrations`) and ensure `000X_*.py` goes into `core/StreamingApp/migrations/`.
- Image handling: `User.save()` resizes profile images. If changing image sizes or validators, update `validate_image_size` and `User.save` in `models.py`.

Security and runtime notes
- Do not hardcode `SECRET_KEY` or DEBUG in code. `core/settings.py` reads from `.env`; keep that pattern. Be mindful that `DEBUG` controls template error detail.
- The app uses SQLite by default (`db.sqlite3`). For production, point `DATABASES` to a proper RDBMS and update migrations.

Examples (copyable snippets for agents)
- Lookup a Series by UUID safely:

```py
from django.shortcuts import get_object_or_404
from core.StreamingApp.models import Series
series = get_object_or_404(Series, series_id=series_id)
```

- Persist watch progress (server-side behavior is in `save_progress`): send JSON POST to the endpoint (URL name: `save_progress` in `StreamingApp/urls.py`) with body {"episode_id": "<uuid>", "progress_seconds": 123} and include session cookie / CSRF token.

What not to change without running migrations or manual checks
- Any changes to UUID primary keys, field names used in templates (e.g., `thumbnail_picture`, `profile_picture_url`, `video_id`) or context names like `series`/`episode` require updating templates and possibly adding migrations.

Files to inspect first when asked for a change
- `core/StreamingApp/models.py` — data model and image handling
- `core/StreamingApp/views.py` — main web logic, endpoints, role guards, save_progress
- `core/StreamingApp/form.py` — forms for create/edit flows
- `core/templates/` — templates are tightly coupled to context variables (search here when UI changes requested)
- `core/settings.py` — environment & media/static config

If something's missing or ambiguous
- If instructions require build/test commands not in the repo, ask for the developer's preferred Python environment (venv/pyenv/conda) and whether there's a `requirements.txt`. The project expects `django`, `Pillow`, and `django-environ` at minimum.

Please review this file and tell me which parts you'd like expanded (examples, commands, or a checklist for PRs). I'll iterate. 

---
> Source: [Qiuraa/web-streaming-uas](https://github.com/Qiuraa/web-streaming-uas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-18 -->
