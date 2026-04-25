# CLAUDE.md — AI Assistant Guide for mail.tm

## Project Overview

A Flask web application for interacting with Mail.tm temporary email accounts. Provides a browser-based inbox viewer with session-based authentication, auto-refresh, message reading/deletion, and email address clipboard copy. Deployable locally or on Vercel.

## Tech Stack

- **Backend:** Python 3.8+ / Flask 3.0.2
- **Frontend:** HTML/CSS/JS with Bootstrap 5.3, jQuery 3.6, Font Awesome 6 (all via CDN)
- **API:** Mail.tm REST API (`https://api.mail.tm`) with JWT authentication
- **Config:** python-dotenv for environment variables
- **Deployment:** Vercel (via `vercel.json` + `api/index.py` entry point)
- **Tests:** unittest with unittest.mock (no pytest)

## Project Structure

```
mail.tm/
├── app.py                    # Main Flask app — MailTM class + routes
├── api/
│   └── index.py              # Vercel serverless entry point (imports create_app)
├── tests/
│   ├── __init__.py
│   └── test_mailtm.py        # Unit tests for MailTM class + Flask routes
├── fetch_mailtm_emails.py    # Standalone email fetch utility
├── requirements.txt          # Python dependencies
├── vercel.json               # Vercel build/routing config
├── .env.example              # Environment variable template
├── .claude/
│   ├── settings.json         # Project-level Claude Code config (permissions, hooks)
│   ├── rules/                # Modular convention files (auto-loaded)
│   │   ├── flask-conventions.md
│   │   ├── frontend.md
│   │   ├── security.md
│   │   └── api-patterns.md
│   └── skills/
│       └── run/SKILL.md      # /run — start the dev server
├── templates/
│   ├── base.html             # Base template (sidebar layout, CDN imports, toast utility)
│   ├── index.html            # Inbox + message viewer
│   ├── login.html            # Login / registration page
│   └── error.html            # Error display page
└── static/
    ├── css/
    │   └── style.css         # Custom styles
    └── screenshot.png        # App screenshot
```

## Setup & Running

```bash
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env  # then edit with real credentials
python app.py         # starts on http://localhost:8000
```

### Environment Variables (.env)

- `FLASK_SECRET_KEY` — Flask session secret (auto-generated if not set)

Note: Mail.tm email/password are entered via the web UI login page, not environment variables.

## Running Tests

```bash
python -m unittest discover tests
```

Tests use `unittest.mock` to mock all `requests` calls to the Mail.tm API. No real API calls are made. The test suite covers:

- **`TestMailTMClass`** — Unit tests for the `MailTM` API wrapper (auth, messages, domains, account operations)
- **`TestFlaskRoutes`** — Integration tests for Flask route handlers using `create_app(mail_client=mock)` injection

## Architecture

### `MailTM` class (`app.py`)

Stateless API wrapper. Static methods for unauthenticated calls (`get_domains`, `create_account`, `authenticate`); instance methods require a `token` for authenticated calls (`get_messages`, `get_message_content`, `delete_message`, `mark_message_as_read`, `get_account`, `delete_account`).

### `create_app()` factory (`app.py`)

Application factory that optionally accepts a `mail_client` parameter for dependency injection in tests. Routes are split into:

- **Auth routes** (`/login`, `/auth/domains`, `/auth/register`, `/auth/login`, `/auth/logout`) — session-based authentication with JWT token stored in Flask session
- **App routes** (`/`, `/message/<id>`, `/refresh`, `/domains`, `/account`) — require authenticated session; return templates for page loads, JSON for AJAX

### Authentication flow

1. User visits `/login` and either registers a new account or logs in with existing credentials
2. JWT token, email, account_id, and password are stored in `session`
3. Authenticated routes use `_get_mail_client()` to build a `MailTM` instance from the session token
4. 401 responses from the API clear the session and redirect to login

## Development Notes

- **No linter/formatter config** — no Black, Pylint, ESLint, or Prettier configured
- **No CI/CD** — no GitHub Actions workflows
- **No Docker** — no Dockerfile or docker-compose
- **Debug mode is on** — `app.run(debug=True)` in `app.py`

## Conventions

Detailed conventions are in `.claude/rules/`. Key points:

- Keep the app as a single `app.py` unless complexity requires splitting
- Frontend deps via CDN (no npm/bundler)
- Use `.env` for secrets; never commit `.env`
- Flask routes return templates for pages, JSON for AJAX
- All Mail.tm API calls go through the `MailTM` class — no raw `requests` calls outside it
- Use `secrets` module for generating passwords/tokens, never `random`
- Sanitize email HTML content before rendering to prevent XSS

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alejandrosejas) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
