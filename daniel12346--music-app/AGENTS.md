<!--
Guidance for AI coding agents working on this repository.
Keep this short, actionable, and specific to the repository layout and conventions.
-->

# GitHub Copilot Instructions

Purpose: Help AI agents be productive quickly in this Next.js + Supabase starter app.

- **Big picture**: This is a Next.js 13 app using the App Router (see `app/`). Authentication and DB access are provided by Supabase via the `@supabase/ssr` helpers. The project follows the server-first App Router pattern (most pages are server components), with explicit client components where needed.

- **Key directories and files**:

  - `app/` — top-level App Router routes and route groups. Notable files:
    - `app/layout.tsx` — root layout, global providers (fonts, theme, audio player provider).
    - `app/actions.ts` — server actions for auth flows (signUpAction, signInAction, signOutAction).
    - `app/(auth-pages)/`, `app/(main)/`, `app/protected/` — route-grouping conventions.
  - `components/` — UI components and shadcn/ui primitives. Example: `components/providers.tsx`, `components/audio-player.tsx` (plus `audio-player.css`).
  - `utils/supabase/` — Supabase helpers:
    - `utils/supabase/client.ts` — browser/client supabase: use in client components.
    - `utils/supabase/server.ts` — server/client used in Server Components, Server Actions, and middleware (uses cookie store).
  - `utils/` — misc helpers (`utils/utils.ts` contains `encodedRedirect` used in server actions).
  - `state/store.tsx` — global client state (uses `zustand`).
  - `database.types.ts` and `lib/database.ts` — DB schema typing and helpers.

- **Auth and session management (important)**:

  - The app uses cookie-backed Supabase sessions via `@supabase/ssr`. Use `createServerClient` (wrapped in `utils/supabase/server.ts`) for server-side requests so cookies propagate correctly.
  - Middleware entry point: `middleware.ts` calls `updateSession` (see `utils/supabase/middleware`). Do not remove the static/image matcher exceptions — they avoid interfering with asset requests.
  - Example: In `app/actions.ts`, server actions call `createClient()` from `utils/supabase/server.ts`.

- **Data flow & patterns**:

  - Server-first rendering: prefer fetching data in Server Components or Server Actions. Use the server client helper (`utils/supabase/server.ts`) when you must access Supabase from server-side code.
  - Client components that need Supabase auth should use `utils/supabase/client.ts` (browser client).
  - For redirects and flash-like messages, the codebase uses `encodedRedirect` in `utils/utils.ts` rather than raw response objects — follow this pattern in new server actions.

- **Dev scripts & local runs**:

  - Start dev server: `npm run dev` (runs `next dev`). Build: `npm run build`. Start production: `npm run start`.
  - Environment variables required: `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`. Tests are not present; do not assume a test runner.

- **Conventions & style specifics**:

  - TypeScript is used everywhere — keep types consistent (`database.types.ts` used for Supabase types).
  - UI primitives follow shadcn/ui conventions. `components.json` is used by shadcn tooling — avoid changing it unless you re-run `shadcn` install steps.
  - Tailwind is used for styling; classes live inline in components and in `globals.css`.
  - Global providers are orchestrated in `components/providers.tsx` and mounted in `app/layout.tsx`.

- **What to avoid / gotchas**:

  - Don’t call the browser `createBrowserClient` from server code — use `createServerClient` (see `utils/supabase/server.ts`).
  - Middleware's matcher excludes static assets — maintain that to prevent unexpected middleware runs.
  - There are route groups (parentheses in folder names). Keep those semantics when moving/renaming routes.

- **Example snippets (follow these exact files)**:

  - Server action pattern: `app/actions.ts` — calls `const supabase = await createClient();` (server helper) and then `supabase.auth.*`.
  - Client component pattern: import `{ createClient }` from `utils/supabase/client.ts` and use it in components with `"use client"`.

- **If you change env or runtime behavior**:
  - Add any new runtime env names to top-level README and mention where they are consumed (e.g., `app/layout.tsx` reads `VERCEL_URL` for `metadataBase`).

If anything above is unclear or you'd like additional rules (committing, linting, testing), tell me which area to expand. Also tell me if you want a stricter checklist for pull requests (e.g., required file touches, changelog entries).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Daniel12346) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
