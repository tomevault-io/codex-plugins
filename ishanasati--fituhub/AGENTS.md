# Copilot Instructions

## Project

**Cooked or Capable** — A student productivity web app built with Next.js (App Router) + TypeScript + Tailwind CSS + Supabase.

Features: productivity self-assessment quiz ("Are You Cooked?"), random challenge generator (3 modes), daily streak tracking, and a student dashboard.

## Build & Dev Commands

```bash
npm run dev      # Dev server at localhost:3000
npm run build    # Production build
npm run lint     # ESLint
```

Copy `.env.local.example` to `.env.local` and fill in Supabase credentials before running.

## Architecture

```
app/
  home/          # Landing/hero page (Server Component)
  quiz/          # 20-question quiz with scoring (Client Component)
  challenge/     # Random challenge generator with confetti (Client Component)
  dashboard/     # Stats + weekly graph (Server Component with data fetching)
  profile/       # Badges + quiz history + sign out (Server Component)
  (auth)/login/  # Supabase email auth (Client Component)
  layout.tsx     # Root layout with Navigation
  page.tsx       # Redirects to /home

components/
  Navigation.tsx          # Fixed nav with mobile menu
  ui/ProgressBar.tsx      # Reusable animated progress bar
  ui/Badge.tsx            # Streak achievement badge
  dashboard/WeeklyGraph.tsx  # 7-day bar chart

lib/
  quiz-data.ts       # 20 questions + calculateResult() scoring function
  challenge-data.ts  # 90 challenges across 3 categories + getRandomChallenge()
  supabase/
    client.ts   # Browser Supabase client (createBrowserClient)
    server.ts   # Server Supabase client (createServerClient + cookies)

middleware.ts   # Protects /dashboard, /profile, /challenge, /quiz routes
types/
  database.ts   # Full Supabase DB types (quiz_results, challenges_completed, streaks)
supabase-schema.sql  # Run this in Supabase SQL editor to set up tables + RLS
```

## Supabase Database

Tables: `quiz_results`, `challenges_completed`, `streaks`. All have RLS enabled — users can only access their own rows. Run `supabase-schema.sql` in the Supabase SQL editor to initialize.

## Key Conventions

- **Server vs Client components** — Dashboard and profile pages are Server Components that fetch data directly. Quiz, challenge, and auth pages are `"use client"` (they need interactivity/state).
- **Supabase clients** — Always use `lib/supabase/client.ts` in Client Components and `lib/supabase/server.ts` in Server Components/actions. Never mix them.
- **Streak update logic** — In `challenge/page.tsx`, streak updates check `last_activity_date` against today and yesterday to increment/reset correctly.
- **Quiz scoring** — `calculateResult()` in `lib/quiz-data.ts` maps raw score (20–80) to focus/discipline/distraction percentages. Calibrated so a max-cooked score (80) yields ~32% focus, ~40% discipline.
- **Challenge generator** — Uses a rapid-interval shuffle animation (80ms per tick, 8 ticks) before landing on the final result. Confetti fires on every 3rd completion.
- **Animations** — Use `framer-motion` for page transitions and card animations. `canvas-confetti` for milestone celebrations.
- **Tailwind dark theme** — `bg-[#0a0a0f]` is the base background. White/opacity utilities (`text-white/40`, `border-white/10`, `bg-white/5`) are used throughout for layering.
- **Path aliases** — `@/` maps to project root (e.g., `import { createClient } from "@/lib/supabase/client"`).
- **Environment variables** — `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_ANON_KEY` are required.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/IshanAsati)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/IshanAsati)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
