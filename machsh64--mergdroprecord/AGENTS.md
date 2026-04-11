# Copilot Instructions for AI Agents

## Project Overview
This is a Binance airdrop points tracking system built with Node.js, Express, and PostgreSQL. It records daily trading activities and calculates points for airdrop eligibility. The frontend is served from the `public/` directory, and the backend exposes RESTful APIs for record management and statistics.

## Architecture & Data Flow
- **Backend:**
  - Main entry: `server.js`
  - Database logic: `models/database.js` (PostgreSQL, table: `transactions`)
  - API endpoints:
    - `GET /api/records/:date` — fetch record by date
    - `POST /api/records` — create/update record (upsert by date)
    - `PUT /api/records/:id` — update record by ID (currently upserts by date)
    - `GET /api/stats/:year/:month` — monthly stats
    - `GET /api/records` — all records
    - `GET /api/health` — health check
  - Static files: served from `public/` (default route `/` serves `index.html`)
- **Frontend:**
  - Located in `public/` (HTML, CSS, JS)
  - Interacts with backend via above API endpoints

## Developer Workflows
- **Start server:**
  - `npm start` — runs `server.js`
  - `npm run dev` — runs with `nodemon` for auto-reload
  - Windows: use `start.bat` if present
  - Linux/Mac: use `start.sh` if present
- **Database setup:**
  - Uses PostgreSQL; connection via environment variables (`env.example`)
  - Table auto-creates on server start (see `initDatabase` in `models/database.js`)
  - For manual setup, use `init.sql`
- **Frontend build:**
  - If a `frontend/` directory exists, build with `npm run build:frontend` (not present in current structure)

## Project-Specific Patterns & Conventions
- **Upsert by date:** All record creation/updates are keyed by `date` (unique constraint in DB)
- **API responses:** Always return `{ success, data, message }` (and sometimes `error`)
- **Points calculation:** `net_points = points_balance + points_trading - points_consumed` (see API responses)
- **Error handling:** Centralized error middleware in `server.js`; returns generic message unless in development
- **Environment config:** Use `.env` (see `env.example` for required variables)
- **Static assets:** Place frontend files in `public/`; backend serves them automatically

## Integration Points
- **PostgreSQL:** Required; configure via environment variables
- **Docker:** Use `docker-compose.yml` or `docker-compose.dev.yml` for containerized setup
- **Environment:** Copy `env.example` to `.env` and fill in DB credentials

## Key Files & Directories
- `server.js` — main backend logic and API routing
- `models/database.js` — all DB operations and schema
- `public/` — frontend assets
- `env.example` — environment variable template
- `init.sql` — manual DB schema setup
- `docker-compose.yml` — container orchestration

## Example: Adding a New API Endpoint
- Define route in `server.js`
- Implement DB logic in `models/database.js`
- Follow response format `{ success, data, message }`

---
For questions or unclear conventions, review `server.js` and `models/database.js` for current patterns. Ask for feedback if any workflow or pattern is ambiguous or missing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/machsh64)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/machsh64)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
