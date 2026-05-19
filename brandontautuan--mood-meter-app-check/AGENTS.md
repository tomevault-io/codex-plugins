# Mood Meter App - AI Coding Agent Instructions

## Project Architecture

This is a **monorepo with three independent applications** sharing a root for dev orchestration:

1. **Kiosk Student App** (`kiosk-srcMTapp/`) - Mood entry interface (React + TypeScript + Vite)
2. **Teacher Dashboard** (`dashboard-figma-dashboard/`) - Data visualization dashboard (React + TypeScript + Vite + Recharts)
3. **Backend API** (`server/`) - Express REST API; Supabase-backed in production

**Critical**: Each app has its own `package.json`, `node_modules`, and Vite config. The root `package.json` provides orchestration scripts via `concurrently` plus a small standalone Vite app in `src/` (deployed via `vercel.json`).

## Development Workflow

### Starting the full system
```bash
npm run dev:all
```
This starts all three services (server + kiosk + dashboard) under `concurrently`.

### Individual service commands
```bash
npm run dev:kiosk     # Student kiosk only
npm run dev:dashboard # Dashboard only
npm run dev:server    # Backend only
```

When working in a specific app directory, `cd` into it first (`cd kiosk-srcMTapp` or `cd dashboard-figma-dashboard`), then use local commands like `npm run dev`.

## Data Flow & Core Concepts

### The Two-Level Mood System (L1/L2)
- **L1 (Quadrant)**: High-level mood categorization based on energy and pleasantness
  - `high-pleasant` → Yellow quadrant
  - `high-unpleasant` → Red quadrant
  - `low-pleasant` → Green quadrant
  - `low-unpleasant` → Blue quadrant
- **L2 (Specific Emotion)**: 25 specific emotions per quadrant (100 total)
  - Defined in `kiosk-srcMTapp/data/emotions.ts` as a static list
  - Mapped differently in the dashboard at `dashboard-figma-dashboard/src/utils/emotionCategories.ts`

**Key difference**: Kiosk app uses hyphenated IDs (`high-pleasant`), dashboard uses underscored types (`high_energy_pleasant`).

### Mood Entry Structure
```typescript
{
  id: string;              // e.g., "e_1698345600000"
  timestamp: string;       // ISO 8601 format
  dateOnly: string;        // YYYY-MM-DD
  l1: { id: string; label: string };
  l2: { id: string; label: string };
  timeToSelectMs: number;  // Time tracking feature
}
```

### API Communication
- Kiosk uses `kiosk-srcMTapp/services/api.ts` for submission. Falls back to localStorage if the request fails.
- Direct Supabase submission is enabled when `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY` are set.
- Backend talks to Supabase via `server/supabase-database.js`; legacy file storage lives at `server/data/moods.json`.

## UI & Component Patterns

### Radix UI + Tailwind Utility Pattern
Both apps use shadcn/ui-style components (`components/ui/`) with Radix UI primitives. The `cn()` helper in `components/ui/utils.ts` merges class names.

### Kiosk App: Page-Based State Machine
The kiosk uses a state machine pattern in `kiosk-srcMTapp/App.tsx`:
```typescript
type Page = "welcome" | "mood-meter" | "sub-emotions" | "all-emotions" | "thank-you";
```

## TypeScript Configuration

Root `tsconfig.json` uses TypeScript project references for the kiosk app. Each app has its own `tsconfig.json`.

## Common Tasks

### Adding a new emotion
Edit `kiosk-srcMTapp/data/emotions.ts` and `dashboard-figma-dashboard/src/utils/emotionCategories.ts` (keep synchronized).

### Modifying API endpoints
1. Update route handler in `server/routes/`
2. Update corresponding service function in `kiosk-srcMTapp/services/` or `dashboard-figma-dashboard/src/services/`

### Adding a new chart/visualization
Dashboard uses Recharts. See existing chart components under `dashboard-figma-dashboard/src/components/` for reference.

---
> Source: [brandontautuan/mood-meter-app-check](https://github.com/brandontautuan/mood-meter-app-check) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-19 -->
