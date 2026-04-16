# Smart Expiry Tracker - AI Coding Agent Guide

## Project Overview
Django-based expiry tracking system for groceries and medicines with advanced OCR, barcode scanning, Celery-powered email reminders, and NGO donation coordination for medicine donations in Kerala.

## Architecture & Data Flow

### Core Django Setup
- **Main app**: `expirytracker/tracker/` contains all business logic
- **Settings**: Dual config in [settings.py](../expirytracker/expirytracker/settings.py) - PostgreSQL in production (Render), local dev mode with `IS_RENDER` flag
- **Database**: User-managed `products` table (unmanaged model) for barcode lookups alongside Django-managed tables

### Key Models ([tracker/models.py](../expirytracker/tracker/models.py))
- **Item**: User's tracked items with `days_until_expiry()` and `is_near_expiry()` helper methods
- **UserProfile**: Extends User with `user_type` ('donor'/'ngo'), email/push notification preferences
- **Product**: Unmanaged model mapping to existing PostgreSQL `products` table (barcode → product_name)
- **NGOProfile**: Organization details with `verified` field for admin approval
- **Donation**: Tracks donation requests between donors and NGOs with status workflow
- **PushSubscription**: Stores web push subscription keys (VAPID) for browser notifications

### Computer Vision Pipeline
**OCR Processing** ([tracker/views.py](../expirytracker/tracker/views.py)):
- `correct_ocr_text()`: Heavy text correction for common OCR misreads (5EP→SEP, O→0, etc.)
- Context-aware replacements near date keywords (EXP, BB, MFG)
- EasyOCR 1.7.1 with GPU acceleration (torch required)
- Date parsing with `dateutil.parser` after OCR cleanup

**Barcode Scanning**: PyZbar + OpenCV for real-time webcam scanning

## Critical Developer Workflows

### Local Development Startup
```powershell
# Terminal 1: Django server
python manage.py runserver

# Terminal 2: Redis (required for Celery)
redis-server redis/redis.windows.conf

# Terminal 3: Celery worker
.\start_celery_worker.bat
# Or manually: celery -A expirytracker worker --loglevel=info --pool=solo
```
**Note**: Use `--pool=solo` on Windows to avoid multiprocessing issues

### Database Migrations
- Always check `managed = False` on Product model - do NOT migrate this table
- Custom migrations for NGO system in [tracker/migrations/](../expirytracker/tracker/migrations/)

### Testing OCR/Barcode
- Use [test_ocr.py](../test_ocr.py) and [test_ocr_speed.py](../test_ocr_speed.py) for standalone OCR testing
- Webcam required for live scanning features

## Project-Specific Patterns

### Email Reminders (Celery Tasks)
[tracker/tasks.py](../expirytracker/tracker/tasks.py):
- Task: `send_expiry_reminders()` runs on schedule (django-celery-beat)
- Respects `UserProfile.email_reminders_enabled` preference
- Exponential backoff retry (3 attempts) in `send_reminder_email_with_retry()`
- HTML email template: [templates/emails/expiry_reminder.html](../expirytracker/templates/emails/expiry_reminder.html)

### API Design (REST + SimpleJWT)
[tracker/api_views.py](../expirytracker/tracker/api_views.py):
- JWT auth for mobile clients (Flutter app in `/flutter/`)
- Dual URL structure: web views in `tracker/urls.py`, API at `/api/` prefix
- CORS wide open (`CORS_ALLOW_ALL_ORIGINS = True`) for dev - **tighten for production**

### Push Notifications
- VAPID keys in settings (Web Push protocol)
- [static/js/push-notifications.js](../expirytracker/static/js/push-notifications.js) handles service worker registration
- [register_fcm_token.py](../expirytracker/register_fcm_token.py) for testing

### NGO Donation Workflow
1. Donor marks item for donation via [donate_to_ngo.html](../expirytracker/templates/donate_to_ngo.html)
2. Creates `Donation` record with status='pending'
3. NGO views requests in [ngo_dashboard.html](../expirytracker/templates/ngo_dashboard.html)
4. Admin verifies NGOs via `NGOProfile.verified` field
5. Legacy Gmail-based notifications (manual) - consider upgrading to automated system

## Deployment (Render)

### Configuration
[render.yaml](../render.yaml) defines:
- **Web service**: Gunicorn WSGI server
- **Worker service**: Celery worker for background tasks
- **Redis**: Managed Redis instance for Celery broker
- **PostgreSQL**: Linked database with auto-generated credentials

### Environment Variables
Required in Render dashboard:
- `DATABASE_URL` (auto-set by Render)
- `SECRET_KEY` (generated)
- `RENDER=true` (triggers production config)
- Email credentials: `EMAIL_HOST_USER`, `EMAIL_HOST_PASSWORD`

### Static Files
- Collectstatic runs in build command
- Served from `staticfiles/` (whitenoise or CDN recommended)

## Common Gotchas

1. **Windows Celery**: Must use `--pool=solo` flag (gevent/eventlet won't work)
2. **OCR Models**: First run downloads ~500MB EasyOCR models - ensure disk space
3. **Barcode Table**: Never run migrations on `Product` model - it's external data
4. **HTTPS Required**: SSL server for testing push notifications locally ([HTTPS_SETUP_README.md](../HTTPS_SETUP_README.md))
5. **Date Parsing**: OCR dates need heavy cleanup - use `correct_ocr_text()` before parsing

## Key Files for Context
- Architecture decisions: [README.md](../expirytracker/README.md)
- Feature TODOs: [TODO.md](../TODO.md), [OCR_TODO.md](../OCR_TODO.md), [barcode_scanner_todo.md](../barcode_scanner_todo.md)
- Deployment guides: [RENDER_DEPLOYMENT_README.md](../RENDER_DEPLOYMENT_README.md)
- NGO feature upgrade: [expirytracker/DONATION_UPGRADE_TODO.md](../expirytracker/DONATION_UPGRADE_TODO.md)

## When Editing Code

- **OCR logic**: Extend `correct_ocr_text()` for new date formats, don't create new functions
- **Models**: Check for UserProfile/NGOProfile relationships before modifying User model
- **Tasks**: Add logging to Celery tasks - critical for debugging async failures
- **API**: Maintain both web views and API endpoints for mobile app compatibility
- **Templates**: Bootstrap 5.3 is standard - match existing patterns in [templates/](../expirytracker/templates/)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Varun4213-bibi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
