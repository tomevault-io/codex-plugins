# Copilot Instructions for Mergington High School Activities API

## Overview
This project is a FastAPI-based web API for managing extracurricular activities at Mergington High School. It allows students to view available activities and sign up for them. The API serves a static frontend and exposes endpoints for activity management.

## Architecture & Key Files
- `src/app.py`: Main FastAPI application. Defines all API endpoints, in-memory data, and static file serving.
- `src/static/`: Contains frontend assets (`index.html`, `app.js`, `styles.css`).
- `src/README.md`: Project-specific usage, API, and setup instructions.

## Data Model & Patterns
- Activities are stored in an in-memory Python dictionary (`activities`).
  - Each activity has: `description`, `schedule`, `max_participants`, and a list of `participants` (emails).
- No persistent storage: all data is lost on server restart.
- Activity names are used as unique identifiers in API routes and data.

## API Endpoints
- `GET /activities`: Returns all activities and their details.
- `POST /activities/{activity_name}/signup?email=...`: Adds a student (by email) to an activity's participants list.
  - **Note:** The current implementation does not prevent duplicate signups. To avoid this, check if the email is already in the `participants` list before appending.
- `GET /`: Redirects to the static frontend (`/static/index.html`).

## Running & Developing
- Install dependencies: `pip install fastapi uvicorn`
- Run the app (from `src/`): `uvicorn app:app --reload`
- Access docs at `http://localhost:8000/docs`
- Static frontend is served at `/static/`

## Project Conventions
- All API logic is in a single file (`app.py`).
- Use FastAPI's built-in exception handling for errors (e.g., `HTTPException`).
- Static files are mounted using FastAPI's `StaticFiles`.
- No database or ORM is used; all data is in-memory.

## Example: Preventing Duplicate Signups
```python
if email in activity["participants"]:
    raise HTTPException(status_code=400, detail="Already signed up")
activity["participants"].append(email)
```

## References
- See `src/README.md` for more details and API usage examples.
- Key logic: `src/app.py`

---
Update this file if you add new endpoints, change data structures, or introduce new conventions.

---
> Source: [buiatte-luizalabs/skills-getting-started-with-github-copilot](https://github.com/buiatte-luizalabs/skills-getting-started-with-github-copilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-26 -->
