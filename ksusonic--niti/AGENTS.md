# NITI - Telegram Mini App for Party Events

## Project Architecture

This is a **Next.js 16 (App Router) Telegram Mini App** for discovering and joining party events. The app runs **inside Telegram** as a Mini App, with authentication handled via Telegram's init data system.

### Tech Stack
- **Frontend**: Next.js 16, React 19, TypeScript, Tailwind CSS 4
- **Telegram SDK**: `@telegram-apps/sdk-react` + `@telegram-apps/telegram-ui`
- **Backend**: Next.js API routes with Supabase (PostgreSQL)
- **Auth**: Telegram init data validation (no separate auth system)
- **Tooling**: Biome (lint/format), pnpm, Sentry

### Key Architecture Decisions

**No traditional auth**: Users are authenticated via Telegram init data passed in `x-telegram-init-data` header. The backend validates this with `@tma.js/init-data-node` using `TELEGRAM_BOT_TOKEN`.

**Client-side only app**: Despite using Next.js, this is primarily CSR due to Telegram Mini Apps limitations. The `Root` component shows a loading state until client mounts (`useDidMount` hook).

**Service role Supabase client**: API routes use `createAdminClient()` with `SUPABASE_SERVICE_ROLE_KEY` (bypasses RLS). User identity comes from validated Telegram data, not Supabase auth.

**Mock environment for development**: `NEXT_PUBLIC_ENABLE_MOCKS=true` enables local development outside Telegram. Mock data is injected via `src/mockEnv.ts` and authenticated using `MOCK_HASH_IDENTIFIER = "some-hash"`.

## Critical Developer Workflows

### Development Commands
```bash
pnpm dev              # Development with mocks (NEXT_PUBLIC_ENABLE_MOCKS=true)
pnpm dev:https        # HTTPS dev server for testing in Telegram (use https://127.0.0.1:3000)
pnpm lint:fix         # Run Biome linter and auto-fix
pnpm typegen          # Generate TypeScript types from Supabase schema
```

### Testing in Telegram
1. Run `pnpm dev:https`
2. Visit `https://localhost:3000`, accept SSL warning
3. Submit `https://127.0.0.1:3000` to [@BotFather](https://t.me/botfather)
4. Launch Mini App in Telegram Web (https://web.telegram.org/k/)

### Database Migrations
```bash
# Generate types after schema changes
pnpm typegen
```
Schema location: `supabase/migrations/20251012211720_create_initial_schema.sql`

## Code Conventions & Patterns

### Authentication Pattern (API Routes)
**Always** start API routes with:
```typescript
const authResult = await checkAuthHeader();
if (authResult instanceof NextResponse) {
  return authResult;
}
const userId = authResult.initData.user?.id;
```
See `src/app/api/subscriptions/route.ts` for the canonical pattern.

### Client-Side API Calls
Use `authenticatedFetch()` or `authenticatedFetchJson()` from `src/lib/api-client.ts`:
```typescript
const events = await authenticatedFetchJson<Event[]>("/api/events");
```
These automatically include `x-telegram-init-data` header from Telegram SDK.

### Mock Detection
Backend routes automatically detect mock data via `isMockInitData()` checking for `hash=some-hash`. This bypasses validation in development.

### Lazy User Registration
Users are created in `profiles` table on first subscription (see `POST /api/subscriptions`). Uses `upsert(..., { ignoreDuplicates: true })` to avoid overwriting existing profiles.

### Telegram UI Components
- Use `@telegram-apps/telegram-ui` components (AppRoot, etc.) for native Telegram feel
- Wrap app with `<AppRoot appearance={isDark ? "dark" : "light"}>` in `Root` component
- Platform detection: `["macos", "ios"].includes(lp.tgWebAppPlatform) ? "ios" : "base"`

### Error Handling
- `TelegramEnvErrorBoundary`: Catches Telegram SDK initialization errors
- `EnvUnsupported`: Fallback UI when Telegram environment fails
- Custom event system: `telegram-env-events.ts` dispatches `TELEGRAM_ENV_UNSUPPORTED_EVENT`

### Idempotent API Design
- Duplicate subscriptions return 200 with `"Already subscribed"` message
- Unique constraint violations (`POSTGRES_ERROR_UNIQUE_CONSTRAINT_VIOLATION = "23505"`) treated as success
- Always return updated `participantCount` in subscription responses

## Project Structure

```
src/
├── app/
│   ├── api/                    # Next.js API routes (auth via checkAuthHeader)
│   │   ├── events/route.ts     # GET events with lineup + participant counts
│   │   └── subscriptions/      # POST subscribe/unsubscribe, GET user subscriptions
│   ├── page.tsx                # Main app: EventFeed + ProfilePage with swipe navigation
│   └── layout.tsx              # Root layout with Telegram UI initialization
├── components/
│   ├── Root/                   # Telegram SDK initialization wrapper
│   ├── EventCard.tsx           # Event card with confetti on subscribe
│   └── ui/                     # Reusable UI components (Button, Avatar, etc.)
├── lib/
│   ├── auth-middleware.ts      # checkAuthHeader() for API route auth
│   ├── api-client.ts           # authenticatedFetch utilities
│   ├── supabase.ts             # createAdminClient() with service role
│   ├── mocks/                  # Mock data for development (config.ts, utils.ts)
│   └── constants.ts            # Shared constants (TELEGRAM_INIT_DATA_HEADER)
├── core/init.ts                # Telegram SDK initialization (mounts miniApp, viewport, etc.)
├── mockEnv.ts                  # Injects mock Telegram environment
└── instrumentation.ts          # Sentry + initialization hook
```

## Data Flow

1. **Client loads**: `Root` component waits for client mount, initializes Telegram SDK via `src/core/init.ts`
2. **Mock detection**: If `NEXT_PUBLIC_ENABLE_MOCKS=true` and not in Telegram, `mockEnv()` injects fake environment
3. **API requests**: Components call APIs via `authenticatedFetchJson()`, which attaches `initData.raw()` header
4. **Backend validation**: API routes validate Telegram data with `checkAuthHeader()` → `validate(rawInitData, TELEGRAM_BOT_TOKEN)`
5. **Database access**: Routes use `createAdminClient()` to query Supabase with service role (bypasses RLS)

## Integration Points

- **Telegram Mini Apps SDK**: `@telegram-apps/sdk-react` provides hooks (`useLaunchParams`, `useSignal(miniApp.isDark)`, `initData`)
- **Supabase**: Direct SQL via service role client; no auth integration (Telegram handles identity)
- **Sentry**: Configured in `instrumentation.ts`, tunneled via `/monitoring` route to bypass ad-blockers
- **Vercel**: Deployment platform; Sentry monitors enabled via `automaticVercelMonitors: true`

## Database Schema

Core tables (see `supabase/migrations/`):
- `profiles`: User profiles (id = Telegram user ID)
- `events`: Party events with lineup
- `event_lineup`: DJ sets with time slots
- `event_participants`: User subscriptions (unique constraint on event_id + user_id)

Types are auto-generated via `pnpm typegen` → `src/types/supabase.ts`

## Common Pitfalls

❌ **Don't** use Supabase client-side auth (use Telegram init data)  
❌ **Don't** forget to mark `TELEGRAM_BOT_TOKEN` and `SUPABASE_SERVICE_ROLE_KEY` as secrets  
❌ **Don't** rely on SSR/SSG - Telegram Mini Apps require CSR  
✅ **Do** use `checkAuthHeader()` in every API route  
✅ **Do** handle idempotent operations (subscriptions can be called multiple times)  
✅ **Do** test with `pnpm dev:https` before deploying to Telegram

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ksusonic)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/ksusonic)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
