# Workspace Instructions

These instructions guide AI agents working in the Booking Schedule System repository.

## Documentation Catalog

_Before explaining concepts, refer to these existing documents:_

- **[Setup & Onboarding](docs/SETUP.md)**: Full walkthrough of Supabase setup, migrations, and environment vars.
- **[Architecture & Design](docs/ARCHITECTURE.md)**: Component tree, data flows, and premium design tokens.
- **[API Reference](docs/API_DOCUMENTATION.md)**: Detailed API endpoints and structures.
- **[Backend Details](backend/README.md)**: FastAPI specifics, schema profiles, and mock integrations.
- **[Implementation Summary](docs/IMPLEMENTATION_SUMMARY.md)**: Recent design and component updates.

## Code Style & Stack

- **Frontend**: Next.js 16 (App Router), TypeScript, Tailwind CSS v4, shadcn/ui.
  - Use `premium-styles.ts` for consistent design tokens (soft borders, backdrop blur, rounded buttons).
  - Use staggered animations (80ms delays, GPU-accelerated transforms).
- **Backend**: FastAPI (Python), SQLAlchemy ORM, PostgreSQL (via Supabase).
  - Clean separation by domain in `backend/app/api/` (e.g., `/api/services`, `/api/bookings`).
- **Auth**: Supabase JWT (supports passwordless email magic links).

## Architecture & Project Structure

- **Frontend** relies heavily on feature-based component folders under `components/` (e.g., `admin/`, `booking/`, `auth/`) and route groups in `app/`.
- **Backend** uses 14 core tables with Row Level Security (RLS) enabled.
  - Feature Profiles: Code acts differently based on `FEATURE_SET` (`core` vs `full`).

## Build and Run Commands

### Frontend

```bash
npm install
npm run dev
```

### Backend (Docker Recommended)

```bash
cd backend
docker compose up --build
```

_To reset the database, drop the volume:_

```bash
docker compose down -v
```

### Seeding & Admin (Run inside backend container)

```bash
# Seed roles and generic data
docker compose exec backend python -m app.seed

# One-time admin bootstrap
docker compose exec backend python -m app.bootstrap_admin --email you@example.com --password "ChangeMe123!" --role superadmin --full-name "Initial Admin"
```

## Common Dev Pitfalls

- **Environment Variables**: Frontend uses `.env.local` (needs Supabase/API URLs). Backend uses `backend/.env`.
- **Database Schema Updates**: If the database gets out of sync, ensure migrations (like `scripts/005_add_email_verification.sql`) are run, or reset the docker volume completely.
- **User Roles Not Applied**: Update user metadata directly in the Supabase Dashboard to include `{"role": "admin"}` if issues arise during manual testing.
- **Payment Setup**: ABA Payway is mocked in development. Webhook secrets are required for production but not dev.
- **Port Conflicts**: Ensure frontend is port 3000, backend is 8000. CORS in backend is configured to allow `http://localhost:3000`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Aicser-Platform)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Aicser-Platform)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
