# HOCKIA Codebase Instructions

## Architecture Overview

HOCKIA is a hockey player recruitment platform with a React 19 frontend (`client/`) and Supabase backend (auth, database, edge functions, storage).

### Frontend Structure (`client/src/`)
- **`pages/`** - Route-level components (Landing, Dashboard variants, MessagesPage, etc.)
- **`components/`** - Reusable UI components with barrel export at `index.ts`
- **`features/`** - Feature modules (admin portal, chat-v2) with own api/hooks/pages
- **`lib/`** - Core utilities: Supabase client, auth store, toast/notifications, utils
- **`hooks/`** - Custom React hooks (useChat, useNotifications, useProfileStrength)
- **`types/`** - TypeScript definitions; `database.types.ts` is auto-generated

### Backend Structure
- **`supabase/functions/`** - Edge Functions (delete-account, notify-vacancy, admin-actions)
- **`supabase/migrations/`** - Database migrations
- **`supabase_setup/`** - Consolidated SQL scripts for fresh project setup (run in order 001-008)

## Key Patterns

### State Management
Zustand stores in `lib/` handle global state:
```typescript
// Auth state - lib/auth.ts
import { useAuthStore } from '@/lib/auth'
const { user, profile, signOut } = useAuthStore()

// Other stores: useToastStore, useNotificationStore, useUnreadStore
```

### Path Aliases
All imports use `@/` prefix configured in vite.config.ts:
```typescript
import { supabase } from '@/lib/supabase'
import { Button, Avatar } from '@/components'
import { useAuthStore } from '@/lib/auth'
```

### Supabase Client
Single typed client instance in `lib/supabase.ts`. Environment variables support both `SUPABASE_*` and `VITE_SUPABASE_*` prefixes.

### Component Conventions
- Components use Tailwind with HOCKIA brand colors (`hockia-primary`, `dark-bg`, etc.)
- Glassmorphism UI pattern with dark mode default
- Export components via `components/index.ts` barrel file
- Lucide icons for all iconography

### Admin Functions
Admin RPC functions aren't in generated types yet. Use the helper pattern in `features/admin/api/adminApi.ts`:
```typescript
const adminRpc = supabase.rpc.bind(supabase) as unknown as (fn: string, params?: Record<string, unknown>) => Promise<{ data: any; error: any }>
```

## Messaging/Chat System

The chat system in `hooks/useChat.ts` handles real-time messaging with these key patterns:

- **Realtime subscriptions** - Supabase realtime on `messages` table, debounced via `VITE_CONVERSATION_REALTIME_DEBOUNCE_MS`
- **Cursor-based pagination** - Uses `(sent_at, id)` cursor for infinite scroll loading older messages
- **Idempotency keys** - Prevents duplicate sends; unique index on `messages.idempotency_key`
- **Optimistic UI** - Messages show immediately with `Sending` → `Sent` → `Read` delivery states
- **Scroll controller** - `useChatScrollController` maintains scroll position when loading older messages and auto-scrolls on new incoming
- **Draft persistence** - `lib/messageDrafts.ts` saves unsent message text per conversation

Conversation participants are normalized: `participant_one_id < participant_two_id` (enforced by trigger).

## Storage Buckets

Four public storage buckets with RLS policies enforcing owner-based access:

| Bucket | Table | Used By | Path Convention |
|--------|-------|---------|-----------------|
| `avatars` | `profiles.avatar_url` | All roles | `{userId}/{filename}` |
| `gallery` | `gallery_photos` | Players/Coaches | `{userId}/{filename}` |
| `club-media` | `club_media` | Clubs | `{clubId}/{filename}` |
| `player-media` | (legacy) | Players | `{userId}/{filename}` |

Policies: Anyone can SELECT (public buckets). INSERT requires `auth.role() = 'authenticated'` with path matching `auth.uid()`. UPDATE/DELETE requires `owner = auth.uid()`.

## RLS Security Patterns

Row Level Security is enabled on all tables. Key patterns in `supabase_setup/003_rls_policies.sql`:

- **Own-data access** - `auth.uid() = user_id` for profiles, gallery, history
- **Participant-based** - Conversations/messages check `participant_one_id` or `participant_two_id = auth.uid()`
- **Admin bypass** - `is_platform_admin()` function checks `app_metadata.is_admin` JWT claim
- **Rate limiting** - Triggers like `enforce_profile_comment_rate_limit()` cap actions (5 comments/24h)
- **Optimistic locking** - `version` column + `increment_version()` trigger prevents concurrent overwrites

When writing queries, always filter by the current user's ID to leverage RLS indexes.

## Profile Strength Calculation

Three role-specific hooks calculate profile completion percentage with weighted buckets:

**`useProfileStrength`** (Players) - 5 buckets:
- Basic Info (25%): nationality, base_location, position
- Profile Photo (20%): avatar_url
- Highlight Video (25%): highlight_video_url
- Journey (15%): ≥1 `playing_history` entry
- Media Gallery (15%): ≥1 `gallery_photos` entry

**`useClubProfileStrength`** (Clubs) - 4 buckets:
- Basic Info (35%): nationality, base_location, year_founded, contact
- Club Logo (25%): avatar_url
- Club Bio (20%): club_bio
- Photo Gallery (20%): ≥1 `club_media` entry

**`useCoachProfileStrength`** (Coaches) - Similar to players with coaching-specific fields

To add a new bucket: add to the `buckets` array with `id`, `label`, `weight`, `completed` boolean, and `action` for navigation.

## Developer Commands

### Frontend (`client/` directory)
```bash
npm run dev          # Start Vite dev server (localhost:5173)
npm run build        # Production build
npm run typecheck    # TypeScript check without emit
npm run lint         # ESLint
npm run test         # Vitest unit tests
npm run test:e2e     # Playwright E2E (public pages)
npm run test:e2e:smoke  # Full smoke suite with auth
```

### Supabase (from repo root)
```bash
supabase db push                    # Apply migrations
supabase functions deploy <name>    # Deploy edge function
supabase gen types typescript --linked > client/src/lib/database.types.ts  # Regenerate types
```

## E2E Testing

E2E tests require real email credentials in `.env.local`:
```bash
E2E_PLAYER_EMAIL=yourname+e2e-player@gmail.com
E2E_PLAYER_PASSWORD=...
E2E_ALLOW_WRITES=1  # Enable write operations
E2E_ALLOWED_SUPABASE_URL_REGEX='^http://localhost:54321$'  # Safety gate
```

Test file naming conventions:
- `*.spec.ts` - Public/unauthenticated tests
- `*.authenticated.spec.ts` / `*.player.spec.ts` - Player session tests
- `*.club.spec.ts` - Club session tests

## User Roles

Three distinct roles with role-specific dashboards:
- **player** - `PlayerDashboard.tsx`, gallery photos, playing history
- **coach** - `CoachDashboard.tsx`, similar to player with coaching-specific fields
- **club** - `ClubDashboard.tsx`, create vacancies, club media, manage applicants

Admin access controlled via `app_metadata.is_admin` JWT claim.

## Notification System

Two isolated notification paths in edge functions:
- `notify-test-vacancy` - Only processes test accounts, sends to hardcoded recipients
- `notify-vacancy` - Only processes real accounts, queries actual users

This prevents test data from ever reaching real users.

## Critical Files

- [client/src/lib/auth.ts](client/src/lib/auth.ts) - Auth state, session handling, profile fetching
- [client/src/lib/supabase.ts](client/src/lib/supabase.ts) - Supabase client setup, type exports
- [client/src/App.tsx](client/src/App.tsx) - Route definitions, lazy loading setup
- [client/src/hooks/useChat.ts](client/src/hooks/useChat.ts) - Core messaging logic
- [client/tailwind.config.js](client/tailwind.config.js) - HOCKIA brand colors
- [supabase_setup/](supabase_setup/) - Database schema reference (run 001-008 for fresh setup)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TianUrien)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/TianUrien)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
