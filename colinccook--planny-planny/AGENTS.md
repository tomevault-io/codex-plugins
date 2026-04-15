# Planny Planny — AI Instructions

## Testing Requirements

All new functionality **must** be covered by appropriate tests:

1. **BDD End-to-End Tests** (preferred): Write Gherkin `.feature` files in `tests/features/` with step definitions in `tests/steps/`. Use Playwright + playwright-bdd.
2. **Unit Tests**: For pure logic (utilities, computations), unit tests with Vitest are acceptable.
3. **All tests must build and pass** before a PR can be merged.

## Tech Stack

- **Frontend**: React 19 + Vite + TypeScript
- **Styling**: Tailwind CSS (mobile-first)
- **Backend**: Supabase (Auth, Postgres, RLS, Realtime)
- **State**: TanStack Query + Supabase Realtime (WebSocket subscriptions)
- **Routing**: React Router v7
- **BDD Tests**: Playwright + playwright-bdd (Gherkin features)
- **CI**: GitHub Actions with local Supabase instance

## Architecture

- **Realtime-first**: After authentication, all data flows through Supabase Realtime WebSocket subscriptions. Auth is the only REST-only flow.
- **Writes**: Go through Supabase REST API (for RLS enforcement), with optimistic UI updates.
- **Reads**: Initial load via REST, then Realtime subscriptions push all changes.
- **RLS**: Row-Level Security on all tables. Check policies in `supabase/migrations/`.

## Project Structure

- `src/components/` — React components organized by feature (auth, calendar, ingredients, settings, layout)
- `src/hooks/` — Custom React hooks (useAuth, useHousehold, useMealPlans, useIngredients)
- `src/lib/` — Supabase client, query helpers, Realtime subscription manager
- `src/pages/` — Page-level components
- `src/types/` — TypeScript types (database schema types)
- `supabase/migrations/` — Database migrations (ordered)
- `tests/features/` — Gherkin BDD feature files
- `tests/steps/` — Playwright BDD step definitions
- `tests/support/` — Test fixtures and helpers

## Branching & Pull Requests

- **Always work in a new branch** — never commit directly to `main`. Create a descriptive feature branch (e.g., `feat/number-stepper`, `fix/swipe-dismiss`) before making changes.
- **Create a Pull Request** for every change. Push the branch and open a PR against `main`.
- **Screenshots are required** for any PR that adds or changes UI functionality. Capture at mobile viewport (390×844) using Playwright or browser dev tools.
- Use the PR template at `.github/PULL_REQUEST_TEMPLATE.md`.
- Include a Screenshots section with images showing the new/changed views.
- **Verify CI passes** — after pushing, poll the PR's check runs (lint, test) until they complete. If any check fails, read the job logs, fix the issue, and push again. Do not consider the task done until all checks are green.

## Conventions

- Mobile-first design: always design for small screens first, then add breakpoints for larger ones.
- Use Tailwind utility classes, avoid custom CSS.
- Use TypeScript strict mode. No `any` types.
- Supabase types are in `src/types/database.ts` — regenerate with `supabase gen types typescript`.
- Each household is isolated: ingredients, meal plans, settings are household-scoped.
- Users can belong to multiple households.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/colinccook) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
