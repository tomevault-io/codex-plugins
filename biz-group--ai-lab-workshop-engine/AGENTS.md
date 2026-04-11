# Workshop Copilot — AI Coding Agent Instructions

## Architecture Overview

Next.js 16 App Router application for running live facilitator-led AI workshops. Supabase provides Postgres, Auth (facilitators only), and Realtime (live sessions). Participants authenticate via custom JWT tokens—not Supabase Auth.

**Key directories:**
- `src/app/admin/` — Facilitator dashboard (server components, Supabase Auth-protected)
- `src/app/s/[sessionId]/` — Participant session view (JWT-protected)
- `src/app/session/[sessionId]/` — Facilitator presenter mode (Supabase Auth-protected)
- `src/app/api/admin/` — Admin CRUD routes (Supabase Auth + org membership check)
- `src/app/api/sessions/`, `src/app/api/submissions/` — Participant routes (custom JWT auth)
- `src/lib/supabase/` — Three Supabase client variants (see below)
- `src/lib/types/database.ts` — All database entity types with Insert/Update variants
- `supabase/migrations/` — Sequential SQL migrations (001–009)

## Two Auth Systems

1. **Facilitators**: Supabase Auth (password login). Middleware in `src/middleware.ts` guards `/admin/*` and `/presenter/*` by checking `sb-*-auth-token` cookies.
2. **Participants**: Custom HS256 JWT via `jose` library (`src/lib/utils/session-token.ts`). Token stored in `workshop_session_token` cookie. No Supabase Auth involvement.

## Supabase Client Rules

| Client | Import | Use in | RLS |
|--------|--------|--------|-----|
| Browser `createClient()` | `'@/lib/supabase'` (barrel) | Client components, Realtime subscriptions | Yes |
| Server `createClient()` | `'@/lib/supabase/server'` (direct) | Server components, admin auth checks | Yes |
| `createServiceClient()` | `'@/lib/supabase/server'` (direct) | API route data mutations, participant routes | **No** (bypasses RLS) |

**Critical**: Never import from `'@/lib/supabase/server'` in client components — it uses `next/headers`. The barrel `'@/lib/supabase'` only exports the browser client.

## API Route Patterns

All API routes return `{ success: boolean, error?: string, data?: T }`.

**Admin routes** (`src/app/api/admin/*`):
```typescript
const supabase = await createClient();                    // cookie-based auth check
const { data: { user } } = await supabase.auth.getUser(); // verify Supabase session
const serviceClient = await createServiceClient();         // switch to service role for data ops
// verify facilitator_users membership, then use serviceClient for all queries
```

**Participant routes** (`src/app/api/submissions/`, etc.):
```typescript
const token = request.headers.get('Authorization')?.replace('Bearer ', '');
const session = await verifySessionToken(token);           // custom JWT verify
const serviceClient = await createServiceClient();         // always service role (no Supabase Auth)
```

Use **Zod** for request body validation (`safeParse`); return 400 with first error message on failure.

## Component Conventions

- **Server components** (default): async functions, fetch data with `await supabase.from(...).select(...)`, use `redirect()` for auth guards
- **Client components** (`'use client'`): receive initial data via props, mutate via `fetch('/api/...')`, never write to DB directly
- **Realtime**: `supabase.channel().on('postgres_changes', ...).subscribe()` in `useEffect` with `supabase.removeChannel()` cleanup
- **Styling**: Tailwind CSS with `cn()` helper from `'@/lib/utils'` (wraps `clsx` + `tailwind-merge`)
- **UI imports**: `import { Button, Card, Modal } from '@/components/ui'` (barrel export)
- **Icons**: `lucide-react`
- **Notifications**: `react-hot-toast`
- **No external state library** — plain `useState`/`useCallback`

## Data Model & Session Lifecycle

Sessions follow: `draft` → `published` → `live` → `ended`

When a session is created from a template, the template's modules/steps/prompt_blocks are **snapshot-copied** into `session_snapshot_*` tables. This allows template edits without affecting running sessions. Snapshots store both `content_markdown_raw` (original with `{VARIABLE}` placeholders) and `content_markdown` (after substitution).

**Variable substitution** (`src/lib/utils/variables.ts`): Uses `{ORG_NAME}`, `{INDUSTRY}`, `{TONE_NOTES}`, `{USE_CASE_1}`–`{USE_CASE_5}` placeholders, resolved from the organization record at session creation.

## Import Conventions

```typescript
// Types — always from barrel
import { Session, Participant, ApiResponse } from '@/lib/types';

// Utils — barrel (excludes session-token which is server-only)
import { cn, formatDate, generateJoinCode } from '@/lib/utils';

// Session token — direct import (server-only, uses next/headers)
import { createSessionToken, verifySessionToken } from '@/lib/utils/session-token';
```

## Development Commands

```bash
npm run dev          # Start dev server
npm run build        # Production build
npm run test         # Vitest (jsdom environment)
npm run test:watch   # Vitest watch mode
npm run db:migrate   # supabase db push
npm run db:seed      # npx tsx scripts/seed.ts
```

## Database Migrations

Sequential SQL files in `supabase/migrations/` (001–009). Add new migrations with the next number prefix. Run with `supabase db push` or paste into SQL Editor. RLS policies are defined across multiple migration files — check existing policies before adding new ones.

## Adding New Features Checklist

1. **Types**: Add to `src/lib/types/database.ts` with `Interface`, `InterfaceInsert`, `InterfaceUpdate` pattern
2. **Migration**: New SQL file in `supabase/migrations/` with RLS policies
3. **API route**: Under `src/app/api/` — use admin auth pattern or participant JWT pattern as appropriate
4. **Components**: Server component for data fetching → client component for interactivity
5. **Supabase join helpers**: Use `getJoinField()` / `getJoinObject()` from `src/lib/utils/supabase-join.ts` when querying joined relations (Supabase returns joins as array or object inconsistently)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Biz-Group)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Biz-Group)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
