Project: A Shadow Within (Next.js + Sanity + Shopify + Spotify)

Concise context for AI coding agents working in this repo:

- Big picture: This is a Next.js 15 (app router) site that uses Sanity as the CMS, Shopify Storefront API for merch, Spotify/YouTube APIs for music metadata and embeds, Klaviyo for newsletter signups, and Gmail (OAuth2) + Nodemailer for contact form email delivery.

- Key folders:
  - `app/` — Next.js app-router pages, layouts and client components. Entry: `app/layout.tsx` and `app/page.tsx`.
  - `app/api/` — Next.js serverless route handlers. Examples: `app/api/fetchSpotify/route.ts`, `app/api/subscribe/route.ts`, `app/api/contact/route.ts`, `app/api/youtube-subs/route.ts`.
  - `sanity/` — Sanity studio config, schemas and helper libs (`sanity/lib/*`, `sanity/env.ts`). See `sanity/plugins/SpotifyFetcherInput.tsx` that calls `/api/fetchSpotify`.
  - `app/lib/` — small API wrappers: `spotify.ts`, `shopify.ts`, `sanity.ts`, and utilities like `videoCover.ts`.

- Environment variables and conventions:
  - Public vs private: anything prefixed with `NEXT_PUBLIC_` is intended for client use. API keys without `NEXT_PUBLIC_` must be kept server-side (for example `SPOTIFY_CLIENT_SECRET`, `GMAIL_REFRESH_TOKEN`, `KLAVIYO_PRIVATE_API_KEY`, `YOUTUBE_API_KEY`).
  - Sanity required env: `NEXT_PUBLIC_SANITY_PROJECT_ID`, `NEXT_PUBLIC_SANITY_DATASET`, `NEXT_PUBLIC_SANITY_API_VERSION` (`sanity/env.ts`).
  - Shopify uses `SHOPIFY_STORE_DOMAIN` and `SHOPIFY_STOREFRONT_ACCESS_TOKEN` in `app/lib/shopify.ts`.

- Patterns and conventions to preserve when making changes:
  - App Router async server components are used (e.g. `app/page.tsx` fetches data with `await` directly). Prefer server components for data fetching when possible.
  - Small client components live under `app/components` and use `"use client"` (e.g. `SignupForm.tsx`, `YouTubeSubscribe.tsx`). Keep data-fetching server-side and expose lightweight endpoints in `app/api/`.
  - Caching: Next headers and `next: { revalidate }` are used throughout. Respect `revalidate` values and `no-store` for `/api/*` endpoints where freshness matters.
  - Image handling: `next.config.ts` lists allowed remote patterns (cdn.sanity.io, img.youtube.com, shopify domains). Use `next/image` and ensure remote hostnames match config.
  - Sanity: the project uses `next-sanity` and `@sanity/image-url` helpers (`sanity/lib/*`). Use these helpers rather than constructing URLs manually.

- Common tasks & commands (developer workflows):
  - Local dev: `npm run dev` (Next.js dev server).
  - Build: `npm run build` then `npm start` to serve the production build.
  - Type checking / linting: TypeScript strict mode enabled; run your editor's typecheck. There is no project-wide linter config in repo root.

- Files that show important behavior and examples to consult when making changes:
  - Server routes: `app/api/fetchSpotify/route.ts` (Spotify token flow + JSON shape), `app/api/contact/route.ts` (Gmail OAuth2 + Nodemailer), `app/api/subscribe/route.ts` (Klaviyo list API), `app/api/youtube-subs/route.ts` (YouTube v3 statistics).
  - Sanity plugin: `sanity/plugins/SpotifyFetcherInput.tsx` shows how the Sanity studio calls the `/api/fetchSpotify` endpoint and expects a specific JSON shape to prefill release documents.
  - Shopify wrapper: `app/lib/shopify.ts` — GraphQL POST pattern with Storefront token and `next.revalidate` usage.
  - Sanity helpers: `sanity/lib/image.ts`, `sanity/lib/live.ts`, and `sanity/lib/client.ts` (studio vs site clients).

- Error handling & edge cases to be careful about:
  - Secrets exposure: never move server-only env values into client code. If you need a value client-side, add `NEXT_PUBLIC_` explicitly and be aware it will be public.
  - API shapes: `/api/fetchSpotify` returns { title, releaseDate, coverImage, spotifyUrl } — plugins depend on these keys. Changing them is a breaking change unless you update the Sanity plugin too.
  - Rate limits: external APIs (Spotify, YouTube, Shopify) are rate-limited — keep server-side caching (`revalidate`) and avoid calling them from client components directly.

- When modifying or adding endpoints:
  - Prefer server (route) handlers in `app/api/` and return JSON using NextResponse. Follow existing patterns for status codes and error shapes.
  - If an endpoint will be called by the Sanity studio (as with Spotify fetcher), preserve the query param interface and response keys.

- Small examples to follow (copy-and-adapt):
  - Fetch Spotify metadata server-side: see `app/api/fetchSpotify/route.ts` for token exchange and response shape.
  - Send newsletter signup: POST to `/api/subscribe` with { email } (see `app/api/subscribe/route.ts`).

Only include code changes that preserve these project conventions; if you need to change an integration shape or env name, add migration notes in the commit message and update the matching Sanity plugin or client wrapper.

If anything above is unclear or you'd like more detail for a particular integration (Shopify queries, Sanity schemas, or OAuth setup), tell me which area and I'll expand the instructions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/davidbowick)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/davidbowick)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
