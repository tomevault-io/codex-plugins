# Overmarksgården Intra – Copilot Instructions

## Project Overview
Danish internal intranet app for Overmarksgården (residential facility). Provides meal plans, shared calendar/activities, and user profiles for residents and staff.

**Stack**: Next.js 16 (App Router) · Tailwind CSS v4 · Supabase (auth + db) · Framer Motion · Capacitor (Android) · PWA

**Key Principle**: ALL content visible to residents (app) is exclusively controlled by admin/staff through the website. Residents have read-only access to shared content.

## Architecture

### Route Structure
- **Public pages** (`src/app/`): Home, `/madplan`, `/faelleskalender`, `/profil`, `/login`
- **Admin section** (`src/app/admin/`): Protected dashboard for staff/admin roles
- **API routes** (`src/app/api/`): Server-side endpoints (e.g., `admin/create-staff`)

### Supabase Integration
- **Browser client**: `src/lib/supabase/browser.ts` – singleton pattern, use `createSupabaseBrowserClient()`
- **Server client**: `src/lib/supabase/server.ts` – async, use `await createSupabaseServerClient()`
- **Tables**: `profiles`, `meals`, `activities`, `house_status`, `private_events`, `activity_signups`, `room_qr_codes`, `weekly_summaries`
- **Auth**: Email/password via Supabase Auth; roles stored in `profiles.role`

### Historical Data & Weekly Navigation
Both Madplan and Fælleskalender use **week-based navigation** for residents:
- Navigate forward/backward through weeks
- "Gå til denne uge" button to return to current week
- Week number displayed (ISO week)
- All historical data preserved for browsing

Admin Kalender uses **month-based navigation** for better overview when managing activities.

### AI Summaries (Future)
Table `weekly_summaries` prepared for AI-generated content:
- Types: `weekly`, `monthly`, `biannual`, `yearly`
- Fields: `week_number`, `year`, `content`, `ai_generated`, `highlights` (JSONB)
- Plan: Generate inspirational summaries every 6 months

### Data Access Control (RLS)
Admin/staff control all shared content; residents have read-only access:
| Table | Residents | Staff/Admin |
|-------|-----------|-------------|
| `meals`, `activities`, `house_status` | Read-only | Full CRUD |
| `profiles` | View all, edit own | Edit any |
| `private_events`, `activity_signups` | Own only | Own only |
| `room_qr_codes` | Verify active | Full CRUD |

### Realtime Subscriptions
All tables are in `supabase_realtime` publication. App pages subscribe to updates:
```tsx
const channel = supabase
  .channel("meals-changes")
  .on("postgres_changes", { event: "*", schema: "public", table: "meals" }, () => fetchData())
  .subscribe();
```
Updates by admin on website are instantly visible in the app.

### User Roles
Roles stored in English, displayed in Danish:
| DB Value | Danish Label | Access |
|----------|--------------|--------|
| `resident` | Beboer | Read-only |
| `staff` | Personale | Admin panel (no user management) |
| `admin` | Administrator | Full access |

Use `isStaffOrAdmin(role)` from `src/types/user.ts` for permission checks.

## Key Patterns

### UI Components
- **`AnimatedCard`**: Glassmorphism cards with Framer Motion. Respects `prefers-reduced-motion`.
- **`DynamicBackground`**: Time-of-day gradient backgrounds (morning warm, night dark).
- **`BottomNav`**: Mobile navigation bar, hidden on `/login`.

### Styling Conventions
- Tailwind v4 with `@import "tailwindcss"` in `globals.css`
- CSS variables: `--font-geist-sans`, `--font-geist-mono`
- Mobile-safe padding utilities: `pb-safe`, `pt-safe`, `pb-nav` (for bottom nav clearance)
- Gradients use `bg-linear-to-*` (Tailwind v4 syntax)

### Motion & Accessibility
All animated components check `useReducedMotion()` from Framer Motion. Example:
```tsx
const prefersReducedMotion = useReducedMotion();
const variants = prefersReducedMotion ? { initial: {}, animate: {} } : { /* full animation */ };
```

### Client Components
Pages fetching data use `"use client"` directive with `useEffect` for data loading. Pattern:
```tsx
"use client";
import { createSupabaseBrowserClient } from "@/lib/supabase/browser";
// ...
useEffect(() => {
  const supabase = createSupabaseBrowserClient();
  // fetch and subscribe to realtime
}, []);
```

## Development Commands

```bash
npm run dev          # Local dev server (Turbopack, dynamic)
npm run build        # Production build (static export for Capacitor)
npm run lint         # ESLint

# Android (Capacitor)
npm run android:run  # Build → sync → run on device/emulator
npm run android:sync # Sync web files to Android project
npm run android:open # Open in Android Studio
```

## Environment Setup
Create `.env.local` with:
```
NEXT_PUBLIC_SUPABASE_URL=...
NEXT_PUBLIC_SUPABASE_ANON_KEY=...
```

## Admin Features

### Dashboard (`/admin`)
Displays stats cards showing counts from `meals`, `activities`, `profiles` tables. Links to sub-pages.

### Madplan (`/admin/madplan`)
Weekly meal plan editor with week navigation. CRUD operations on `meals` table:
- Fields: `date`, `title`, `description`, `vegetarian_option`, `allergens[]`
- Allergens stored as array, input as comma-separated string

### Kalender (`/admin/kalender`)
Monthly activity management. CRUD operations on `activities` table:
- Fields: `title`, `description`, `location`, `starts_at`, `ends_at`, `category`, `requires_signup`, `max_participants`
- Categories: `sport`, `social`, `creative`, `health`, `education`, `other`
- Category colors defined in `categoryColors` object

### Brugere (`/admin/brugere`)
User management with role editing. Only `admin` role can access user management features. Roles editable via modal form.

### Beboer Aftaler (`/admin/aftaler`)
Staff/admin management of private events for ALL residents (40 people):
- Table: `private_events` with `user_id`, `title`, `description`, `starts_at`, `ends_at`, `all_day`
- Filters: Month navigation, resident selector (dropdown), search by name/title/room
- Staff/admin can create, edit, delete events for any resident
- Events grouped by date with resident name and room number displayed
- Modal form with resident picker, datetime inputs, all-day toggle
- Used for appointments (lægebesøg, tandlæge, etc.) managed by staff

### QR-koder (`/admin/qr-koder`)
Room-based QR login system for residents:
- Table: `room_qr_codes` with `room_number`, `code`, `resident_name`, `is_active`
- QR encodes URL: `{origin}/login?code={code}`
- Codes are 8-char alphanumeric (no ambiguous chars like 0/O, 1/I/L)
- **Auto-generation**: When a resident is assigned a room in `/admin/brugere`, a QR code is auto-created if none exists
- Shows resident name linked via `profiles.room_number`
- Green highlight for rooms with assigned residents
- Regenerating deactivates old code and creates new one (for resident changeover)
- Print single or "Print all" for batch printing
- Login flow: Scan QR → verify code → create/sign in as `room{X}@overmark.local`

## Language & Localization
- UI text is in **Danish** (weekday names, greetings, labels)
- Danish date formatting: `toLocaleDateString("da-DK", {...})`
- Role labels: See `getRoleLabel()` in `src/types/user.ts`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ArneDjurhuus)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/ArneDjurhuus)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
