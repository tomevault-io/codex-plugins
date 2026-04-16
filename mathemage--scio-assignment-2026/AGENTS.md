# Copilot Instructions

## Project Overview

This is a **Student Progress Monitor** — a real-time web application for teachers to monitor student learning progress during group activities. Teachers create groups, students join via QR codes, and everyone communicates through real-time chat with WebSocket support.

The project is licensed under AGPL-3.0-or-later.

## Tech Stack

| Layer | Technology |
|-------|------------|
| Backend | Python 3.9+ with FastAPI |
| Frontend | React 18 with TypeScript and Vite |
| Database | SQLite with SQLAlchemy ORM |
| Auth | Google OAuth2 + JWT tokens |
| Real-time | WebSockets |
| QR Codes | react-qr-code |

## Project Structure

```
backend/                    # FastAPI Python backend
├── main.py                # App entry point, CORS config
├── database.py            # SQLAlchemy models
├── schemas.py             # Pydantic validation schemas
├── auth_utils.py          # JWT token utilities
├── dependencies.py        # Auth middleware/dependencies
├── requirements.txt       # Python dependencies
├── .env.example           # Environment config template
└── routers/
    ├── auth.py            # OAuth & authentication endpoints
    ├── groups.py          # Group management endpoints
    └── chat.py            # WebSocket & chat endpoints

frontend/                   # React + TypeScript SPA
├── package.json
├── vite.config.ts
├── tsconfig.json
└── src/
    ├── App.tsx            # Router setup
    ├── main.tsx           # Entry point
    ├── pages/             # Page components (Dashboard, GroupPage, etc.)
    ├── hooks/             # Custom hooks (useAuth)
    ├── services/          # API service layer (api.ts)
    ├── types/             # TypeScript interfaces
    └── utils/             # Helper functions

scripts/                    # Helper automation scripts
├── list_users.py          # List all users in DB
├── set_user_role.py       # Change user roles
├── make_teacher.sh        # Quick teacher role assignment
└── quick_demo.sh          # Demo environment setup
```

## Build & Run Commands

### Backend

```bash
cd backend
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

### Frontend

```bash
cd frontend
npm install
npm run dev          # Start dev server on port 3000
npm run build        # Production build (tsc && vite build)
npm run type-check   # TypeScript type checking (tsc --noEmit)
```

## Coding Conventions

### Backend (Python)

- Use **FastAPI** routers to organize endpoints by domain (`routers/auth.py`, `routers/groups.py`, `routers/chat.py`).
- Define all request/response models as **Pydantic schemas** in `schemas.py`.
- Use `datetime` type in Pydantic schemas for temporal fields to match SQLAlchemy `DateTime` columns.
- Define database models with **SQLAlchemy ORM** in `database.py`.
- Use **dependency injection** for authentication (`dependencies.py`).
- JWT `sub` claim must be a string — convert `user.id` to string when creating tokens.
- All new endpoints must include proper authorization checks.

### Frontend (TypeScript/React)

- Use **functional components** with React hooks.
- Define TypeScript interfaces in `src/types/`.
- Use the `apiService` from `src/services/api.ts` for all backend API calls.
- The `ApiService` automatically injects the auth token from `localStorage` as a fallback.
- All `fetch` requests must include `credentials: 'include'` for cross-origin session cookies.
- Use **React Router v6** for navigation, with routes defined in `App.tsx`.
- Use `useAuth` hook from `src/hooks/useAuth.tsx` for authentication state.

## Security Guidelines

- Never commit secrets or credentials (`.env` files are gitignored).
- Use environment variables for all configuration (Google OAuth credentials, SECRET_KEY).
- Backend enforces role-based access control (Teacher/Student roles).
- Input validation is handled by Pydantic on the backend.
- SQLAlchemy ORM prevents SQL injection.
- The `DEMO_ALLOW_SELF_ROLE_CHANGE` env var must default to `false` in production.

## Authentication Flow

1. User clicks "Sign in with Google" → redirects to `/auth/google`.
2. Backend redirects to Google OAuth.
3. Google redirects back to `/auth/google/callback`.
4. Backend creates/finds user, generates JWT.
5. Redirects to frontend with token.
6. Frontend stores token in `localStorage`.
7. All API requests include token in `Authorization: Bearer <token>` header.
8. Frontend fetches user data on page load via `GET /auth/me` — refreshing the browser is sufficient to see role changes.

## Testing

There is no automated test suite. Testing is done manually:

- Use the interactive API docs at `http://localhost:8000/docs`.
- Use helper scripts in `scripts/` for user/role management.
- Run `python scripts/verify_requirements.py` for automated requirement checks.
- Run `npm run type-check` in the frontend to validate TypeScript types.

## Git and GitHub

- Create a new Issue, then create a new Branch, and finally a new PR.
- Use best practices for commit messages. Every commit message should follow this regex:
  `(?:fix|chore|docs|feat|refactor|style|test)(?:\(.+\))?: [A-Z].+(?:\s#\d+)?`
- Use keywords with scopes every time.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mathemage) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
