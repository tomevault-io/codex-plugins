# Copilot Instructions for Ufirst-Admin

## Project Overview
This is the **Ufirst Admin Dashboard** - a Next.js 15 web application that serves as the administrative interface for the Ufirst mobile app. The project uses modern React 19 with TypeScript and Tailwind CSS v4.

## Architecture & Structure
- **Monorepo pattern**: Main application lives in `mainapp/ufirst/` (not project root)
- **Next.js App Router**: Uses the modern `src/app/` directory structure
- **Component-first**: React Server Components by default, Client Components when needed
- **TypeScript strict mode**: All files must be properly typed

# Copilot Instructions for Ufirst-Admin

This file gives AI coding agents concise, actionable knowledge to be productive in this repo.

Overview
- The main web app lives in `mainapp/ufirst/` (monorepo). Work there for installs, builds and tests.
- It's a Next.js (v15) App Router project using React 19 + TypeScript + Tailwind CSS v4.

Quick commands (run from `mainapp/ufirst`)
```powershell
npm install
npm run dev        # dev server (Turbopack)
npm run build      # production build (Turbopack)
npm run start      # start built app
***
# Copilot Instructions — Ufirst-Admin (concise)

Purpose: quick, actionable guidance so AI coding agents become productive immediately.

- Repo layout: monorepo. The primary app is `mainapp/ufirst/` — work and run commands from there.
- Frameworks: Next.js (App Router, v15), React 19, TypeScript (strict), Tailwind CSS v4.

Quick commands (run from `mainapp/ufirst`):
```powershell
npm install
npm run dev        # dev server (Turbopack) -> http://localhost:3000
npm run build
npm run start
npm run lint
npm run test
npm run test:coverage
```

Key patterns & files (use as authoritative examples):
- `src/app/` — App Router pages. Add new pages under `src/app/<feature>/` (server components by default).
- `src/app/layout.tsx` — global layout (fonts, providers).
- `src/app/globals.css` — Tailwind + CSS custom properties (theme tokens). Avoid adding other global CSS files.
- `src/lib/` — shared helpers and hooks (e.g., `src/lib/auth.ts`, `src/lib/utils.ts`).
- `src/config/api.ts` — central API configuration. Update this when changing backend endpoints.
- `src/lib/api/clients.ts` and `src/lib/api/trainers.ts` — canonical patterns for API calls used by UI and tests.
- `pages/api/*` — Next.js API routes used for local integration and mirrored in tests (use these when creating server-side behaviors used in tests).
- `tests/` — Jest tests. See `jest.config.js` and `jest.setup.js` for environment setup and `jest-fetch-mock` usage.

Conventions & small rules to follow here:
- Server components are the default. Add `'use client'` only when a component needs hooks/state or browser-only APIs.
- Path alias: imports use the `@/*` alias (maps to `./src/*`) — update `mainapp/ufirst/tsconfig.json` if you change it.
- Styling: prefer Tailwind utilities + component-scoped classes; use CSS custom properties in `globals.css` for theme tokens.
- Images: prefer `next/image` for optimization.

Testing & debugging specifics:
- Tests run from `mainapp/ufirst` with `npm run test`. Coverage output: `mainapp/ufirst/coverage/` (lcov + HTML).
- Tests isolate network calls using `jest-fetch-mock`. Look at `mainapp/ufirst/jest.setup.js` for mocks and global setup.
- API routes under `pages/api/*` are reachable during dev at `/api/*` and are referenced directly by tests — keep their signatures stable.

Developer workflows and expectations:
- Feature work: add UI pages under `src/app/dashboard/` and reusable logic to `src/lib/` or `src/config/`.
- When adding APIs used by tests, prefer editing `pages/api/*` (these emulate server behavior in CI/local).
- Linting & formatting: run `npm run lint` before committing; follow TypeScript strict types.

Examples (common edits):
- Add a new dashboard page: create `src/app/dashboard/feature/page.tsx` (server component). If interactive, add `src/app/dashboard/feature/ClientComponent.tsx` with `'use client'`.
- Add an API helper: extend `src/lib/api/clients.ts` with typed request/response shapes and update `src/config/api.ts` if endpoints change.

Notes for AI agents:
- Prefer minimal, focused changes that follow existing patterns. Match import aliases (`@/`) and the server/component split.
- Reference the tests in `mainapp/ufirst/tests/` for examples of expected behavior and mocking.
- If changing build or test scripts, update `mainapp/ufirst/package.json` and document the change here.

If anything here is unclear or you want the CI/deployment details added, tell me which area to expand.

***

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TunnelVision-Org)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/TunnelVision-Org)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
