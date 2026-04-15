# Copilot Instructions for Travel Dashboard

This guide enables AI coding agents to work productively in this Next.js 14 dashboard project. It summarizes architecture, workflows, and conventions unique to this codebase.

## Architecture Overview

- **Framework:** Next.js 14 (App Router), React 19, TypeScript, Tailwind CSS v4, Zustand for state management.
- **App Structure:**
  - `src/app/`: Entry point, layout, and main page (`layout.tsx`, `page.tsx`).
  - `src/components/`: UI components (e.g., `DestinationCard`, `StatsBar`, `FiltersBar`, `SearchBar`, `ThemeToggle`, `InitTheme`).
  - `src/data/`: Static data (`destinations.ts`).
  - `src/lib/`: Utility modules (`filter.ts`, `sort.ts`, `constants.ts`).
  - `src/stores/`: Zustand stores for travel and theme state.
  - `public/images/`: Destination images referenced by data.

## Data Flow & State

- **Destinations:**
  - Data is loaded from `src/data/destinations.ts` into Zustand store (`useTravelStore`).
  - Filtering, sorting, and search are performed in-memory in the main page (`page.tsx`).
- **Theme:**
  - Managed via `useThemeStore` and persisted in `localStorage`.
  - `InitTheme` component syncs theme on load; `ThemeToggle` updates theme and DOM class.

## Component Patterns

- **Client Components:** All interactive components use `"use client"` directive.
- **Props:** Components receive state and callbacks via props, e.g., `DestinationList`, `FiltersBar`, `SearchBar`.
- **State Management:** Use Zustand stores for global state (`useTravelStore`, `useThemeStore`).
- **Styling:** Tailwind CSS classes are used throughout. Dark mode is toggled via `.dark` class on `<html>`.

## Developer Workflows

- **Install & Run:**
  ```bash
  npm install
  npm run dev
  ```
- **Build:**
  ```bash
  npm run build
  ```
- **Lint:**
  ```bash
  npm run lint
  ```
- **No built-in tests.** Add tests in `src/` if needed.

## Conventions & Patterns

- **Imports:** Use `@/` alias for `src/` (see `tsconfig.json`).
- **Sorting/Filtering:** Use utility functions in `src/lib/` or inline logic in `page.tsx`.
- **Progress Bar:** Calculated in `StatsBar` from store data.
- **Mark as Visited:** Toggled via store method, reflected in UI.
- **Responsive UI:** All components use Tailwind for responsiveness.

## External Integrations

- **No backend/API calls.** All data is local/static.
- **Image paths:** Must match files in `public/images/`.

## Key Files

- `src/app/page.tsx`: Main dashboard logic (state, filtering, sorting, rendering).
- `src/stores/useTravelStore.ts`: Travel destination state logic.
- `src/components/`: UI and interaction patterns.
- `src/lib/`: Utility logic for sorting/filtering.
- `README.md`: Quickstart and tech stack summary.

---

**For updates:**

- Merge new conventions and workflows here.
- Keep instructions concise and project-specific.

---

_Ask for feedback if any section is unclear or missing important project-specific details._

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KristinaKiparoidze) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
