# Project Guidelines

## Project Overview
- Mona Mayhem is an Astro v5 app with a Node adapter in server mode.
- Primary app code lives in `src/pages/` (file-based routes) and `src/pages/api/` (API routes).
- Treat `workshop/` as training material, not implementation guidance for production changes.

## Build And Dev
- Install dependencies: `npm install`
- Start dev server: `npm run dev`
- Build production output: `npm run build`
- Preview production build: `npm run preview`

## Astro Best Practices
- Prefer Astro file-based routing patterns in `src/pages/` over custom routing abstractions.
- Keep API handlers in `src/pages/api/**` using typed `APIRoute` exports.
- For dynamic API routes, use bracket params (example: `src/pages/api/contributions/[username].ts`).
- Keep API responses explicit: set status codes and JSON `Content-Type` headers.
- Preserve SSR setup in `astro.config.mjs` (`output: 'server'` with `@astrojs/node` adapter) unless a task explicitly requires changing runtime mode.
- Keep TypeScript strict-compatible changes (project extends `astro/tsconfigs/strict`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mchithna)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/mchithna)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
