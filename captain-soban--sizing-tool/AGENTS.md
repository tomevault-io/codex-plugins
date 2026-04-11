

# Copilot Instructions for sizing-tool

## Project Overview

This is a real-time agile story point sizing tool for sprint planning, built for agile teams. It uses SvelteKit (2.x), Svelte 5 (with runes API: `$state`, `$effect`), TypeScript, shadcn-svelte UI, and Tailwind CSS. The app is client-only (no backend), using localStorage for persistence. It supports anonymous/named voting, multiple point scales, and session/round management for planning poker.

**Target audience:** Agile teams, Scrum masters, Product owners.  
**Business goal:** Streamline sprint planning and estimation with a fast, modern, and privacy-respecting tool.

## Technology Stack

- SvelteKit 2.x, Svelte 5 (runes API)
- TypeScript
- shadcn-svelte UI (slate base color)
- Tailwind CSS (custom color theme: red, white, blue; gray buttons, black text)
- Vite for build tooling
- Vitest (unit tests), Playwright (E2E)

## Architecture & Data Flow

- **Source Structure:**
   - `src/lib/` – shared utilities, services, Svelte components (UI, modals, toggles, etc.)
   - `src/routes/` – SvelteKit pages for session, admin, and settings flows
   - `src/lib/localStore.ts` – localStorage abstraction for session/user data
   - `src/lib/services/` – business logic (e.g., roundService, participantModeService)
   - `src/lib/api/sessionClient.ts` – session state management and real-time updates
- **UI:**
   - shadcn-svelte components, Tailwind CSS for all styling
   - Standard layout: settings/exit in top-right, session info in top-left
- **Session Model:**
   - 8-digit session codes (no '0' or 'o')
   - Owner/host and participant roles
   - Point scale: Fibonacci, T-shirt, or custom (see `storyPointOptions`)
   - Voting state, rounds, and results tracked in localStorage
- **Real-Time Updates:**
   - Simulated via local event listeners and polling (no server/SSE)
   - `SessionClient` manages updates, heartbeats, and participant tracking
   - `isConnected` field is defined for participants but not yet fully implemented (see `notes.md` for TODO)

## Developer Workflows

- **Start Dev Server:** `npm run dev`
- **Build:** `npm run build`
- **Preview:** `npm run preview`
- **Unit Tests:** `npm run test` or `npm run test:unit`
- **E2E Tests:** `npx playwright test` or `npm run test:e2e`
- **Lint/Format/Check:** `npm run lint`, `npm run format`, `npm run check`
- **Docker (optional):** `npm run docker:up`, `npm run docker:down`, etc. (see `CLAUDE.md`)
- **Database scripts:** `npm run db:setup`, `npm run db:reset` (for admin/dev only)

## Project-Specific Conventions

- **Session Codes:** Always 8 chars, exclude '0' and 'o' (see code in `sessionClient`)
- **Voting:**
   - Anonymous mode supported (no names required)
   - Man-days mode: if `storyPointOptions` is `['man_days']`, show numeric input
   - Keyboard shortcuts: 0-8, ? for quick voting
- **UI:**
   - Use shadcn-svelte for all new UI; fallback to Svelte 5 if needed
   - Use Tailwind for all styling
   - Place settings/exit in top-right, session info in top-left
   - Color theme: red, white, blue; gray buttons, black text; white for work area
- **Data Privacy:**
   - All data is local; no network calls or external APIs
   - No user tracking or analytics
- **Component Patterns:**
   - Modals (e.g., `NewRoundModal`, `ShareableLinkModal`) use `isOpen` prop and event callbacks
   - Participant/round state managed via Svelte stores and services

## Key Files & Examples

- `src/routes/+page.svelte` – landing page, session creation/join
- `src/routes/session/[sessionCode]/+page.svelte` – main session UI, voting logic, round management
- `src/lib/localStore.ts` – localStorage logic
- `src/lib/services/roundService.ts` – round/estimate logic
- `src/lib/components/` – reusable UI (modals, toggles, stats, etc.)

## Features & TODOs

**Completed:**
- Session management (8-digit codes, recent sessions, quick rejoin)
- Observer mode (toggle between voting/observer)
- Participant stats, shareable links, round management
- Admin dashboard, database monitoring (for dev/admin only)

**Current TODO:**
- Implement `isConnected` field for real-time participant status (see `notes.md`)
- Help icon/text for each page
- Performance optimization for large teams
- Mobile app considerations

## Testing

- Test session code generation, point scale selection, and localStorage persistence
- Validate voting flows (anonymous/named, man-days, custom scales)
- E2E: simulate multiple participants, round transitions, and observer/host roles

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/captain-soban)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/captain-soban)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
