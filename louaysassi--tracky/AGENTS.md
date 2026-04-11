<!-- Copilot instructions tailored for the Tracky (Personal-Finance-Tracker) repo -->
# Copilot / AI Agent Instructions — Tracky

This file contains short, actionable guidance to help an AI coding agent be immediately productive working on Tracky.

- Project type: React + TypeScript frontend (Vite) + Node/Express backend. Frontend built to `dist/` and served by Express from `server/index.js`.
- Dev workflow: `npm run dev` runs both backend (`node server/index.js`) and Vite frontend concurrently. For isolated work use `npm run dev:frontend` or `npm run dev:backend`.

- Key files to inspect before editing:
  - `server/index.js` — Express server, static file serving, mounts API routes, opens browser on Windows.
  - `server/database.js` — Creates/initializes SQLite DB at %APPDATA%/Tracky/tracky.db using `better-sqlite3` and enables foreign keys.
  - `server/routes/data.js` and `server/routes/settings.js` — All backend API endpoints for data and settings.
  - `src/api/client.ts` — Single source of truth for API base URL and all client-side requests — update when adding/changing endpoints.
  - `src/App.tsx` — App bootstrapping, data-loading, debounced save (500ms), payday logic and routing.
  - `src/types.ts` — Central TypeScript types used across components.
  - `README.md` — Contains helpful developer and packaging instructions (desktop shortcut, build steps).

- Database & data conventions (important):
  - The `monthly_data` table stores a JSON blob in the `data` TEXT column. Do not assume normalized columns — edits should serialize/deserialize the entire month object.
  - `settings` is a single-row table (id = 1) containing `default_salary`. Code expects row to exist; `database.js` inserts a default if missing.
  - Backend uses `better-sqlite3` with explicit transactions for bulk saves. Use prepared statements and `db.transaction(...)` when writing many rows.

- API / client change checklist (follow this when modifying endpoints):
  1. Add/modify route file in `server/routes/` and export a router.
  2. Mount the router in `server/index.js` (e.g., `app.use('/api/foo', require('./routes/foo'))`).
  3. Update `src/api/client.ts` with matching client calls and error handling.
  4. Update `src/types.ts` to keep TypeScript interfaces aligned.
  5. Update README if the public behavior or scripts change.

- Frontend patterns to follow:
  - The app keeps a single `allMonthsData` object in `App.tsx` and persists whole-object changes back to the API on a 500ms debounce. Avoid implementing granular server-side columns for monthly items without migrating the DB format.
  - Payday flow: App automatically prompts if date >= 25 and `lastPaydayConfirmed` isn't set for the month. See `App.tsx` for example logic.
  - IDs for bills/transactions/goals are often generated using `Date.now()` or similar — avoid breaking assumptions about unique ID shapes.

- Packaging & platform notes:
  - The server auto-opens a browser with a Windows-specific `start` command. Be mindful when running on non-Windows platforms — the `start` call may need adjusting.
  - The DB lives in the user's AppData (Windows). Any schema changes must include a migration plan and user-backup instructions (see README backup section).
  - There are helper scripts for desktop packaging: `create-desktop-shortcut.bat`, `launch-tracky.bat`, and `build-exe.js` — inspect before modifying packaging behavior.

- Safety and data concerns for agents:
  - Never delete or reset the real DB during development without explicit user consent. Recommend operating on a disposable DB copy when testing migrations.
  - Inform the user if a change will force data migrations or require manual backup steps.

- Quick commands (copyable):
  - Install deps: `npm install`
  - Dev (frontend + backend): `npm run dev`
  - Build frontend: `npm run build` (outputs `dist/`)
  - Start production server: `npm start` (serves `dist/` on port 3000)

- When opening a PR, include:
  - A short note on DB changes (schema, migrations, sample backup/restore steps).
  - Files changed in both backend routes and `src/api/client.ts` for endpoint changes.
  - Any updates to `src/types.ts` and a brief note explaining runtime compatibility with existing DB JSON blobs.

If anything above is unclear or you want coverage of additional hotspots (UI components, analytics calculations, packaging details), say which area and I will expand this file.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LouaySassi)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/LouaySassi)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
