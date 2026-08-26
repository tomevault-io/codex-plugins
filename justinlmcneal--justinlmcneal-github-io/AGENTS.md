# Copilot Instructions — Karry Kraze

## Project Overview
E-commerce store (karrykraze.com) — vanilla HTML/CSS/JS + Tailwind CDN, Supabase backend, GitHub Pages hosting.

## Tech Stack
- **Frontend**: Vanilla JS (ES modules), Tailwind CSS (CDN), no build step
- **Backend**: Supabase (Postgres, Auth, Storage, Edge Functions)
- **Edge Functions**: Deno/TypeScript, deployed via `echo y | npx supabase functions deploy <name> --project-ref yxdzvzscufkvewecvagq`
- **Hosting**: GitHub Pages (JustinlMcNeal/justinlmcneal.github.io)
- **Payments**: Stripe

## Database Rules
- `products.shipping_status` CHECK constraint: only `'mto'` or `NULL` allowed
- Categories: headwear, jewelry, bags, accessories, plushies, lego (UUIDs in categories table)
- Storage bucket `products` exists and is public

## Code Patterns
- Admin JS lives in `js/admin/<section>/` with `index.js` as entry point
- Shared modules in `js/shared/`
- Config in `js/config/env.js` (exports SUPABASE_URL, SUPABASE_ANON_KEY)
- Edge function error handling: always use `catch (err: unknown)` with `err instanceof Error ? err.message : String(err)`
- Edge function imports: use `https://esm.sh/@supabase/supabase-js@2` (Deno-style)

## Principles
- No React, no build tools, no bundlers — keep it vanilla
- Every edge function must handle CORS preflight (OPTIONS → 200)
- Test changes before marking complete
- Git commit with descriptive messages, push to main

---
> Source: [JustinlMcNeal/justinlmcneal.github.io](https://github.com/JustinlMcNeal/justinlmcneal.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-26 -->
