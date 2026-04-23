# Planit — Claude Code Context

> Modern planning application for organizing personal and professional goals.
> CS 3754 Cloud Software Development · 5-person team project (Wulff, Breslow, Abu Harb, Abuhaija, Khawaja).
> Claude Code should read this file in full before writing any code.

---

## Project overview

Planit is a web-based responsive planning application that helps people organize personal and professional goals. Users can create plans, set deadlines, track progress through tasks, and receive reminders — all through a clean, motivational interface.

---

## Tech stack (strict — do not substitute)

| Layer | Technology |
|---|---|
| Frontend | React.js (Create React App) |
| Backend | Node.js / Express |
| Database | PostgreSQL |
| Authentication | JWT (cookie-based) + Google OAuth (OpenID Connect) |
| Hosting | AWS or Azure Cloud Infrastructure |

---

## Repository structure

```
planit/
├── backend/
│   ├── src/
│   │   ├── index.js              # Express app entry point
│   │   ├── config/
│   │   │   └── db.js             # PostgreSQL pool
│   │   ├── middleware/
│   │   │   └── auth.js           # JWT authentication middleware
│   │   ├── routes/
│   │   │   ├── auth.js           # Signup, login, Google OAuth, logout
│   │   │   ├── plans.js          # CRUD for plans
│   │   │   ├── tasks.js          # CRUD for tasks within plans
│   │   │   ├── reminders.js      # CRUD for reminders
│   │   │   └── dashboard.js      # Dashboard summary endpoint
│   │   └── services/
│   │       └── reminderService.js # Cron-based reminder checker
│   ├── migrations/
│   │   ├── 001_initial_schema.sql
│   │   └── run.js                # Migration runner
│   ├── package.json
│   └── .env.example
├── frontend/
│   ├── public/
│   │   └── index.html
│   ├── src/
│   │   ├── index.js              # React entry point
│   │   ├── App.js                # Router + layout
│   │   ├── context/
│   │   │   └── AuthContext.js    # Auth state provider
│   │   ├── services/
│   │   │   └── api.js            # Axios API client
│   │   ├── components/
│   │   │   ├── Navbar.js
│   │   │   ├── ProtectedRoute.js
│   │   │   └── PlanModal.js
│   │   ├── pages/
│   │   │   ├── Login.js
│   │   │   ├── Signup.js
│   │   │   ├── Dashboard.js
│   │   │   ├── Plans.js
│   │   │   ├── PlanDetail.js
│   │   │   └── Reminders.js
│   │   └── styles/
│   │       └── App.css
│   └── package.json
└── CLAUDE.md
```

---

## Database schema

### users
- `id` SERIAL PRIMARY KEY
- `email` VARCHAR(255) UNIQUE NOT NULL
- `password_hash` VARCHAR(255)
- `full_name` VARCHAR(255) NOT NULL
- `google_id` VARCHAR(255) UNIQUE
- `avatar_url` TEXT
- `created_at`, `updated_at` TIMESTAMPTZ

### plans
- `id` SERIAL PRIMARY KEY
- `user_id` INTEGER REFERENCES users(id) ON DELETE CASCADE
- `title` VARCHAR(255) NOT NULL
- `description` TEXT
- `deadline` TIMESTAMPTZ
- `status` ('active', 'completed', 'archived') DEFAULT 'active'
- `priority` ('low', 'medium', 'high') DEFAULT 'medium'
- `created_at`, `updated_at` TIMESTAMPTZ

### tasks
- `id` SERIAL PRIMARY KEY
- `plan_id` INTEGER REFERENCES plans(id) ON DELETE CASCADE
- `user_id` INTEGER REFERENCES users(id) ON DELETE CASCADE
- `title` VARCHAR(255) NOT NULL
- `description` TEXT
- `deadline` TIMESTAMPTZ
- `status` ('pending', 'in_progress', 'completed') DEFAULT 'pending'
- `sort_order` INTEGER DEFAULT 0
- `created_at`, `updated_at` TIMESTAMPTZ

### reminders
- `id` SERIAL PRIMARY KEY
- `user_id` INTEGER REFERENCES users(id) ON DELETE CASCADE
- `plan_id` INTEGER REFERENCES plans(id) ON DELETE CASCADE (nullable)
- `task_id` INTEGER REFERENCES tasks(id) ON DELETE CASCADE (nullable)
- `remind_at` TIMESTAMPTZ NOT NULL
- `message` TEXT NOT NULL
- `is_sent` BOOLEAN DEFAULT FALSE
- `created_at` TIMESTAMPTZ

---

## API endpoints

### Auth
- `POST /api/auth/signup` — email/password registration
- `POST /api/auth/login` — email/password login
- `POST /api/auth/google` — Google OAuth login
- `POST /api/auth/logout` — clear session
- `GET /api/auth/me` — get current user

### Plans
- `GET /api/plans` — list user's plans (with task counts)
- `GET /api/plans/:id` — get plan with tasks
- `POST /api/plans` — create plan
- `PUT /api/plans/:id` — update plan
- `DELETE /api/plans/:id` — delete plan (cascades to tasks)

### Tasks
- `GET /api/tasks?plan_id=:id` — list tasks for a plan
- `POST /api/tasks` — create task
- `PUT /api/tasks/:id` — update task
- `DELETE /api/tasks/:id` — delete task

### Reminders
- `GET /api/reminders` — list upcoming (unsent) reminders
- `POST /api/reminders` — create reminder
- `DELETE /api/reminders/:id` — delete reminder

### Dashboard
- `GET /api/dashboard` — summary stats, overdue plans, upcoming deadlines, reminders

---

## Environment variables

```bash
# Database
DATABASE_URL=postgresql://postgres:password@localhost:5432/planit

# JWT
JWT_SECRET=

# Google OAuth
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=

# Server
PORT=3001
FRONTEND_URL=http://localhost:3000
```

---

## Code quality requirements

- All async operations wrapped in try/catch with user-facing error messages
- JWT stored in httpOnly cookies (not localStorage)
- All database queries parameterized (no SQL injection)
- CORS configured for frontend origin only
- Passwords hashed with bcrypt
- No secrets in client-side code

---

## What NOT to do

- Do not use class components — functional components and hooks only
- Do not store JWT in localStorage — use httpOnly cookies
- Do not skip error handling on any async operation
- Do not use ORMs — use raw SQL with pg for clarity
- Do not put secrets in frontend code

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DroidRobot) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
