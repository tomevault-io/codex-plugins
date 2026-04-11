# Copilot Instructions for RachelFoods

## Project Overview
- **RachelFoods** is a production-grade, multi-tenant e-commerce platform for traditional food vendors.
- **Architecture:**
  - **Frontend:** Next.js 16 (TypeScript, App Router, Tailwind CSS)
  - **Backend:** NestJS (TypeScript), PostgreSQL (Prisma ORM), Stripe, JWT Auth
  - **Key Modules:** Orders, Payments, Wallet, Coupons, Inventory (Kitchen Refill), Admin Dashboard, RBAC
  - **Data Flow:** Orders and payments are tightly coupled with inventory and wallet modules to enforce business invariants (no oversell, no wallet overdraft, atomic transactions).

## Critical Workflows
- **Backend Dev:**
  - `cd backend && npm install`
  - Configure `.env` (see `.env.example`)
  - `npx prisma migrate deploy` (migrate DB)
  - `npm run start:dev` (dev server)
  - **Tests:** `npm test` (Jest, see `jest.config.js`)
  - **Seed Admin:** `npm run seed:admin` (set `ADMIN_EMAIL`/`ADMIN_PASSWORD`)
- **Frontend Dev:**
  - `cd frontend && npm install`
  - Configure `.env.local` (see `.env.example`)
  - `npm run dev`

## Project-Specific Patterns
- **Oversell Prevention:** Inventory is locked during order creation (see "kitchen refill" logic in backend). Orders are only finalized after seller confirmation.
- **Wallet Safety:** All wallet debits/credits are ACID, logged, and cannot go negative. Refunds are credited to wallet automatically.
- **Dual Payment:** Stripe and Cash on Delivery are both supported. Stripe uses webhooks for payment confirmation; COD is confirmed by seller/admin.
- **RBAC:** Roles (ADMIN, STAFF, BUYER) are enforced via NestJS guards and reflected in API endpoints and UI.
- **Chaos Engineering:** Failure scenarios (concurrent wallet ops, inventory depletion, external service failures) are tested and documented (see `docs/CHAOS_TESTING_PHASE_9C.md`).

## Conventions & Integration
- **Prisma:** All DB changes via migrations (`npx prisma migrate dev`).
- **Testing:** Use Jest for backend (`*.spec.ts` in `test/`).
- **Logging:** Winston logger with correlation IDs.
- **API:** RESTful, versioned under `/api`.
- **Deployment:**
  - Backend: Render/Railway, use `npm run build` and `node dist/src/main.js`.
  - Frontend: Vercel, use `npm run build` and `npm start`.
- **Docs:** See `/docs` for module guides, operational checklists, and phase summaries.

## Key Files & Directories
- `backend/src/` — Main NestJS app (modules, controllers, services)
- `backend/prisma/` — Prisma schema and migrations
- `backend/test/` — Jest tests
- `frontend/` — Next.js app
- `docs/` — Architecture, guides, and operational docs

## Quick Tips
- Always run DB migrations before starting backend.
- Use `.env.example` as template for environment setup.
- For new modules, follow existing NestJS module structure.
- Reference `docs/FEATURE_MATRIX.md` and `docs/MODULE_CATALOG.md` for system capabilities and module boundaries.
- For production, follow `DEPLOYMENT_GUIDE.md` and `PHASE_7_PRE_LAUNCH_CHECKLIST.md`.

---
For more, see the main [README.md](../README.md) and `/docs` for deep dives on architecture and workflows.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rachelfuud)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/rachelfuud)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
