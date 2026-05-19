# Copilot Instructions for AI Agents

## Project Overview
- **Framework:** Next.js (App Router, TypeScript)
- **Domain:** Business accounting, including expense vouchers, bank reconciliation, ledgers, and more.
- **Structure:**
  - `app/` — All routes (API and UI) are colocated. Subfolders (e.g., `dashboard/`, `api/`) represent features or endpoints.
  - `components/` — Shared React components.
  - `lib/` — Client/server utilities (API, auth, permissions, logging, etc.).
  - `prisma/` — Database schema and migrations (Prisma ORM).
  - `public/` — Static assets.
  - `scripts/` — PowerShell scripts for setup and database checks.

## Key Patterns & Conventions
- **API routes:**
  - Located in `app/api/feature/` (e.g., `app/api/expense-vouchers/`).
  - Use role-based access: pass `x-user-role` header for protected endpoints.
- **UI pages:**
  - Located in `app/dashboard/feature/page.tsx`.
  - Use React hooks (`useState`, `useEffect`) for data fetching and state.
  - Forms use local state, submit via fetch to API routes.
- **Accounts/Permissions:**
  - Permission logic in `lib/permissions.ts`, `lib/requirePermission.ts`, etc.
  - Always check user role/permissions for sensitive actions.
- **Expense Vouchers Example:**
  - Data fetched from `/api/expense-vouchers` and `/api/accounts`.
  - Account selection uses filtered dropdowns (see `ExpenseVouchersPage`).
  - Items are managed as an array in state, with add/remove/edit.

## Developer Workflows
- **Start dev server:** `npm run dev` (or `yarn dev`, `pnpm dev`, `bun dev`)
- **Database:**
  - Schema: `prisma/schema.prisma`
  - Migrate: `npx prisma migrate dev`
  - Seed: `node prisma/seed.js`
  - Check: `scripts/check-database.ps1`
- **Environment setup:**
  - Use `scripts/setup-env.ps1` or `scripts/create-env.ps1` for .env files.
- **Debugging:**
  - Use browser/React DevTools for UI.
  - Use console logs for server/API debugging.

## Integration & Data Flow
- **Data flow:** UI (dashboard) → fetches API (app/api) → DB (Prisma)
- **Cross-feature:** Use shared utilities in `lib/` for API calls, permissions, and logging.
- **External:** No external API dependencies by default; all business logic is internal.

## Project-Specific Notes
- **Role/permission checks are critical for all API endpoints.**
- **Naming:** Feature folders and files are lower-case, dash-separated.
- **TypeScript:** All code is typed; define interfaces for API data.
- **No test suite or build step is documented; see scripts/ for setup.

## References
- See `README.md` for Next.js basics.
- See `lib/` for reusable logic.
- See `prisma/schema.prisma` for DB structure.
- See `app/dashboard/expense-vouchers/page.tsx` for a full CRUD UI+API pattern.

---
> Source: [omersajjad909-del/smart-business-accounts](https://github.com/omersajjad909-del/smart-business-accounts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-19 -->
