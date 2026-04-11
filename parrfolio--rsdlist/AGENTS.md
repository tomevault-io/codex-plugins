# RSD Wants Development Guidelines

Auto-generated from all feature plans. Last updated: 2026-02-05

## Overview

RSD Wants is a Record Store Day checklist app for record collectors to browse the current year's RSD special releases, save items to a personal wants list, and check them off as acquired while shopping in-store.

## Active Technologies

- TypeScript (strict mode)
- React 18.3 + Vite + React Router
- Tailwind CSS + shadcn/ui (Radix primitives)
- lucide-react (icons)
- Zustand (UI state)
- TanStack React Query (server state)
- Firebase Auth (Google, Facebook, Email/Password)
- Firebase Firestore (with offline persistence)
- Firebase Cloud Functions (Node 20)
- Firebase Hosting + Emulators
- pnpm (flat project, not monorepo)
- Zod (validation)

## Project Structure

```text
src/
├── components/
│   ├── ui/             # shadcn/ui generated components
│   ├── app/            # App-level composed components
│   ├── theme-provider.tsx
│   └── theme-toggle.tsx
├── hooks/              # React Query hooks, auth, Zustand
├── lib/                # Firebase config, auth helpers, utils
├── pages/              # Route components
├── stores/             # Zustand stores
└── types/              # TypeScript types + Zod schemas
data/
└── rsd/                # Seed JSON files by event
scripts/                # Seed scripts
specs/                  # Feature specifications
```

## Commands

```bash
pnpm install              # Install all dependencies
pnpm dev                  # Start web + emulators + functions watch
pnpm lint                 # ESLint
pnpm typecheck            # TypeScript strict check
pnpm test                 # Run tests (optional)
pnpm seed:rsd             # Populate Firestore with release data
```

## Code Style

- TypeScript strict mode required
- Follow shadcn/ui patterns for UI components
- All server state via React Query hooks
- All UI-only state via Zustand stores
- Optimistic updates for all mutations

## Feature: 001-rsd-wants-app

### Core User Journeys

1. **Browse RSD Releases** - Anonymous users can browse current year releases in a grid, search by artist/title, sort A→Z or Z→A
2. **Sign In & Onboard** - Sign in via Google/Facebook/Email, select preset avatar on first login
3. **Add to Wants** - Signed-in users add releases to personal wants list with instant visual feedback
4. **Manage Wants/Acquired** - Check off items to move from Wanted to Acquired while shopping
5. **Account Management** - View profile, change avatar, sign out

### Key Entities

| Entity  | Description                                                                           |
| ------- | ------------------------------------------------------------------------------------- |
| Release | RSD special release (artist, title, album art, format, release type, label, quantity) |
| User    | Authenticated user (uid, displayName, email, avatarId, authProviders)                 |
| Want    | User's saved release with status (WANTED or ACQUIRED)                                 |

### Firestore Collections

```text
/events/{eventId}            # RSD events (rsd_2025_spring, rsd_2025_fall)
/releases/{releaseId}        # Public releases (read-only to clients)
/users/{uid}                  # User profiles
/users/{uid}/wants/{wantId}   # User's wants (denormalized for fast reads)
```

### Routes

| Route      | Description                 |
| ---------- | --------------------------- |
| `/`        | Redirect to `/rsd`          |
| `/auth`    | Sign in / sign up           |
| `/rsd`     | RSD release list (browsing) |
| `/mylist`  | Wanted + Acquired sections  |
| `/account` | Profile, avatar, sign out   |

### shadcn/ui Components (required)

- button, card, input, badge, tabs, dialog, sheet
- dropdown-menu, avatar, checkbox, scroll-area
- separator, skeleton, toast (or sonner)

### App-Level Composed Components

| Component        | Purpose                                   |
| ---------------- | ----------------------------------------- |
| ReleaseCard      | Card + Button + Badge for release display |
| TopSearchSortBar | Input + DropdownMenu for filtering        |
| BottomTabs       | Mobile navigation tabs                    |
| SectionHeader    | Separator + counts for list sections      |
| EmptyState       | Card + text + CTA for empty lists         |

### Mobile UX Requirements

- Mobile-first responsive design
- Bottom tab navigation on screens < md breakpoint
- Touch targets ≥ 44px
- One-handed operation for My List checkboxes
- Firestore offline persistence for in-store use

### React Query Hooks

```typescript
useRsdReleasesQuery({ year, search, sort }); // Fetch releases
useMyWantsQuery({ year }); // Fetch user wants
useToggleWantMutation(); // Add/remove from wants
useSetWantStatusMutation(); // Toggle WANTED ↔ ACQUIRED
```

### Zustand Store (UI State)

```typescript
interface UIState {
  viewMode: 'GRID' | 'LIST';
  year: number;
  sort: 'ARTIST_ASC' | 'ARTIST_DESC';
  search: string;
}
```

### Security Rules Summary

- Anonymous read allowed for `/rsdReleases`
- Auth required for all `/users` reads/writes
- Users can only access their own data (uid path match)
- Client writes to `/rsdReleases` denied
- Status field validated against WANTED | ACQUIRED enum

## Assumptions (MVP Scope)

- Both Spring + Fall RSD events supported
- Event IDs follow pattern: rsd*{year}*{season} (spring/fall)
- Current year releases only
- Preset avatars only (no upload)
- No per-item notes, priority, or budget tracking
- No social features (sharing, public profiles)
- Curated fallback dataset acceptable if scraping fails

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/parrfolio)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/parrfolio)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
