# Copilot Instructions for DjangoTestBooking

## Project Overview
- This is a Django 5.2.6 project for managing lesson bookings, with a focus on tennis lessons.
- Main app: `bookings` (models, forms, views, templates)
- Project config: `booking_project` (settings, URLs)
- Uses SQLite (`db.sqlite3`) for local development.

## Architecture & Data Flow
- Core models: `Client`, `Booking`, `LessonType` (see `bookings/models.py`)
- Bookings are linked to clients and lesson types; bookings are only allowed for the current month and cannot overlap.
- Views in `bookings/views.py` handle booking creation, calendar display (JSON for FullCalendar), user signup, login, password reset, and cancellation.
- URLs are namespaced under `/bookings/` (see `booking_project/urls.py` and `bookings/urls.py`).
- Templates are in `bookings/templates/bookings/` and use Django's template language.
- Static files (including FullCalendar JS) are in `bookings/static/bookings/`.

## Developer Workflows
- **Run server:** `python manage.py runserver`
- **Migrate DB:** `python manage.py makemigrations bookings` then `python manage.py migrate`
- **Create superuser:** `python manage.py createsuperuser`
- **Run tests:** `python manage.py test bookings`
- **Debug:** Use Django shell: `python manage.py shell`

## Project-Specific Patterns
- Bookings must be for the current month and not in the past (see model/form validation).
- Booking times must be in 30-minute increments and not overlap with existing bookings.
- User signup creates both a Django user and a linked `Client` object.
- Password reset and username recovery are implemented with custom forms and email templates.
- Calendar data is served as JSON for FullCalendar integration (`calendar_view` endpoint).
- All booking actions require authentication except signup and login.

## Integration Points
- FullCalendar JS is used for calendar UI (see static files and `calendar.html`).
- Email backend is set to console for development; change in `settings.py` for production.
- Static files are served from `bookings/static/bookings/`.

## Key Files & Directories
- `bookings/models.py`: Data models and validation logic
- `bookings/views.py`: Main business logic and endpoints
- `bookings/forms.py`: Form validation and custom signup logic
- `bookings/templates/bookings/`: HTML templates for all pages
- `bookings/static/bookings/`: JS/CSS assets, including FullCalendar
- `booking_project/settings.py`: Django settings
- `booking_project/urls.py`: Project-level URL routing

## Example Patterns
- To add a new booking type, update `LessonType` in `models.py` and adjust forms/templates as needed.
- To customize calendar behavior, modify `calendar_view` in `views.py` and related JS in static files.
- For new user flows, extend `SignUpForm` and related views/templates.

---

If any section is unclear or missing, please provide feedback for further refinement.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/isaiaha09)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/isaiaha09)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
