# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is Chance?

Chance is a social party app played on a single shared phone alongside physical board games. When a game event triggers a "Chance" moment, the active player draws a card from a shared pool. Cards carry dares, drinking mechanics, and prompts. The card pool grows as registered players submit cards across sessions. Guests need no account — they join by entering a display name.

## Terminology

| Term | Definition |
|------|------------|
| **Game Session** | A single instance of play. Created by a registered host; expires after 16 days or when the host leaves. |
| **Player** | An ephemeral game identity scoped to one session. Optionally linked to a User. |
| **User** | A permanent registered account (email + password, invite-code-gated). |
| **CardVersion** | Immutable snapshot of card content. Edits create new versions; draw history references the version drawn. |
| **cardType** | `chanceCard` or `reparationsCard` — set at creation, Card-level (not version-level), admin-only to change post-creation. |
| **card_sharing** | Per-player, per-session setting: `none` / `mine` (default). Controls how a registered player contributes cards to the draw pool. |

## Project Status

This app is **released and in production**. The database is live on real devices. Any schema change requires a migration — never modify `init.ts` alone.

## Monorepo Structure

pnpm workspaces + Turborepo.

```
root/
├── apps/
│   ├── backend/        # Next.js 15 App Router — API + admin portal
│   └── mobile/         # Ionic 8 / Capacitor 8 / React 19 / Vite 6 — targets web + native equally
├── packages/
│   └── core/           # Shared Zod schemas, AppError types, constants
├── turbo.json
└── pnpm-workspace.yaml
```

## Common Commands

```bash
# Install all dependencies
pnpm install

# Run all apps in dev mode
pnpm dev

# Build everything (respects Turbo pipeline: core → apps)
pnpm build

# Run tests
pnpm test

# Lint / typecheck
pnpm lint
pnpm typecheck

# Backend only
pnpm --filter backend dev
pnpm --filter backend db:migrate
pnpm --filter backend db:seed

# Mobile only
pnpm --filter mobile dev        # Vite dev server on port 8100
pnpm --filter mobile build
npx cap sync                    # After build, sync to Android/iOS
npx cap run android
npx cap run ios
```

Turbo pipeline order: `build` → `test`, `lint`, `typecheck`. The `core` package must build before apps.

## Code Style

Prettier config (root `.prettierrc`): double quotes, semicolons, trailing commas (ES5), 4-space indent, 100-char print width.

TypeScript strict mode across all packages.

## Shared Core Package (`packages/core`)

Single source of truth for:

- **Schemas** (`src/schemas/`): Zod schemas for all domain entities — `Card`, `Session`, `Player`, `CardTransfer`, `DrawEvent`, `FilterSettings`, `Vote`, `InvitationCode`, `MutationQueueEntry`. Use these for runtime validation AND TypeScript type inference (via `z.infer<>`). Never duplicate types in the apps.
- **Errors** (`src/errors/`): `AppError` base + `AuthenticationError`, `AuthorizationError`, `ValidationError`, `NotFoundError`, `ConflictError`, `InvitationCodeError`. All carry `code`, `message`, and optional `details`.
- **Constants** (`src/constants/`): card weight multipliers, token TTLs, poll intervals, reveal delay (`3000ms`), connection check endpoint, mutation retry limits.

## Backend Architecture (`apps/backend`)

**Stack:** Next.js 15 App Router, better-sqlite3 (synchronous SQLite), TypeScript.

**Request flow:** `middleware.ts` (CORS + security headers) → `app/api/**/route.ts` (Zod validation + `withAuth`/`withAdmin` HOF) → `lib/services/` (business logic) → `lib/repos/` (prepared statements) → SQLite.

**Key patterns:**

- All queries use prepared statements — no string interpolation.
- Boolean columns use `boolToInt`/`intToBool` bridge helpers (SQLite has no native bool).
- Migrations are timestamp-named TypeScript files in `src/db/migrations/`, applied in order by `pnpm db:migrate` (`src/db/migrate.ts`). **Every schema change requires a migration** — `init.ts` only runs on fresh databases.
- `withAuth` HOF wraps protected routes; `withAdmin` additionally checks `users.is_admin`.
- Every API response uses the `{ ok, data/error, serverTimestamp }` envelope — no exceptions.
- Mutations return the full updated entity so the client can reconcile in one step.
- `GET /api/health` — no auth, no DB access; pure liveness probe.

**Card draw algorithm** (`lib/card-picker/`): weighted random at draw time. Base weight `1.0`, session-card boost `3.0×`, upvote bonus `+0.2` per net vote (cap `+2.0`), downvote penalty `0.5×`, recently-drawn suppression `0.1×`. Excluded cards (flagged/removed or failing session filters) are dropped before weight calculation.

**Admin portal** (`app/admin/`): Mantine UI, protected by a separate admin session (not user JWT), at `/admin`.

## Frontend Architecture (`apps/mobile`)

The frontend targets **web browsers and native (iOS/Android) equally**. The web build is a first-class platform — low-friction, no install required. Native Capacitor features (secure storage, network plugin, App state events, keep-awake, QR scanner) are **optional enhancements** with web fallbacks; the app must work in a plain browser without any Capacitor plugin.

**Stack:** Ionic 8, Capacitor 8, React 19, Vite 6, React Router v5, TanStack React Query.

**Route access tiers** — most of the app is reachable with any valid JWT:
- **Open** — no JWT required (`/`, `/login`, `/register`, `/join/:code`)
- **Session member** — any JWT (guest or registered) + active session; redirects to `/` if no session (`/lobby`, `/game`, `/card`, `/submit-card`)
- **Registered only** — non-guest JWT; redirects to `/login` (`/settings`, create-session action)

**Provider stack order** (App.tsx):
`AuthProvider` → `DatabaseProvider` → `SessionProvider` → `CardProvider` → `TransferProvider` → `AppHeaderProvider` → `AppErrorBoundary` → `IonReactRouter`

**State management:**

- Server state: React Query — in-memory only; stale 2min, GC 10min
- Auth / Session / Card / Transfer state: domain Contexts

**Loading & transitions — two tiers, no manual `isLoading` flags:**

- **Queries:** `useSuspenseQuery` everywhere. On first route visit (no cache), the component suspends and the Suspense boundary renders a page-specific skeleton. Background refetches and polls never re-suspend — cached data stays on screen.
- **Mutations:** wrap in React 19 async `startTransition`. Use `isPending` from `useTransition` to show a Mantine `<LoadingOverlay>` over the affected section. The form/page stays rendered underneath — never disabled, inputs preserved for retry.
- **Suspense boundary rule:** boundaries belong at the route level, *inside* `IonReactRouter` and therefore inside all Context providers. A Suspense boundary above a Context unmounts it when children suspend, resetting all state.

**Error handling:** mutations fire directly; on failure the error is shown inline. No retry queue, no optimistic writes. `ApiClient` returns `ApiResult<T>` — never throws. `NetworkStatusBanner` shows when the Capacitor Network plugin reports offline; interactive elements are disabled until reconnected.

**Forms:** all forms use `react-hook-form` (`useForm`) with `zodResolver` from `@hookform/resolvers/zod`. Use shared Zod schemas from `@chance/core` as the resolver base, extending with user-facing messages via `.extend()` where needed. Never use manual `useState` for individual form fields, uncontrolled refs, or ad-hoc validation logic. Per-field validation errors render inline below each input; API/server errors go to `setError("root", ...)` and render near the submit button. `IonInput` fires `onIonInput` not native `onChange` — bridge by calling `register("field").onChange({ target: { value: ... } })` inside `onIonInput`.

**Session polling:** React Query `refetchInterval` — 5s foreground, 30s backgrounded. Background detection uses Capacitor App state events (native) or the Page Visibility API (web). Network status uses Capacitor Network plugin (native) or `navigator.onLine` / `online`/`offline` events (web). Paused while offline; resumes on reconnect. Poll endpoint accepts `?since=<ISO>` to return only changes since last known timestamp.

**Local storage** (all storage paths are abstracted; native uses Capacitor plugins, web uses browser APIs):

- Registered JWT tokens → Capacitor Secure Storage (native) / `sessionStorage` (web)
- Guest JWTs → in-memory only; not persisted on any platform
- Guest player tokens → Capacitor Preferences (native) / `localStorage` (web)
- API URL override → Capacitor Preferences (native) / `localStorage` (web)

**ApiClient** (`lib/api/client.ts`): singleton, 15s timeout via `AbortController`, auto-refreshes tokens on `401`/`X-Token-Status: invalid` and replays once. Returns typed `ApiResult<T>` discriminated union — never throws.

## API Design Conventions

- `withAuth` validates any JWT (guest or registered). Most gameplay endpoints — draw, vote, transfer — only require `withAuth`.
- Registered-only check is applied to: `POST /api/sessions` (create), `POST /api/cards` (submit), and account management. Everything else in gameplay is guest-accessible.
- Guest JWTs are **ephemeral** — session-scoped, never persisted to Capacitor Secure Storage; cleared when the session ends or the app restarts.
- **In-session account claiming** (`POST /api/auth/claim`): a guest can log in or register mid-session. Requires a guest JWT in the `Authorization` header + credentials in the body. Atomically sets `session_players.user_id` and issues a registered JWT pair. All prior guest activity (draws, votes) is preserved. Rejected with `409` if the registered user is already in the session. `/login` and `/register` detect an active guest JWT and route through the claim flow automatically.
- **Single-device play is the primary use case.** A registered host creates the session; other players join as ephemeral guests by entering a display name. All players take turns on the same phone; the active player is selected via an in-session player switcher. Players with their own devices can also join and follow along simultaneously — both modes are supported.
- Invitation codes: single-use, admin-created, optional expiry. `POST /api/auth/register` consumes the code atomically.
- `card_game_tags` — zero rows = universal card (eligible for all sessions). One or more rows = eligible only for sessions filtered to one of those games.
- Cards are deactivated (`card.active = false`) by the owner or admin; this excludes them from all future draw pools but preserves draw history. No hard-delete.
- `card.is_global` (admin-only) promotes a card to the global pool. `card.created_in_session_id` (nullable FK → sessions) tracks the session where the card was born — used for game-lineage pool tier and the 3× boost.
- Only registered users can submit cards. Guests draw only.
- `session_players.card_sharing` (`'none' | 'mine'`, default `'mine'`) controls how a registered player contributes to the session pool.
- Flags are a lightweight report signal only — no automatic hold.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thedinj) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-11 -->
