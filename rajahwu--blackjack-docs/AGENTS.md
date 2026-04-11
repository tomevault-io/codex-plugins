<!-- Auto-generated guidance for AI coding agents working on this repo -->
# Copilot instructions — Blackjack Docs (Astro)

Purpose: help an AI code assistant be productive editing this Astro-based documentation site and the small React/TS feature under `src/pages/game`.

Quick start
- Root commands (run from project root): use pnpm or npm as preferred by developer. package.json scripts:
  - dev: `astro dev` (local dev server)
  - build: `astro build` (produces `dist/`)
  - preview: `astro preview`

Architecture & important files
- Site is an Astro project using React components for interactive parts. Key folders:
  - `src/pages/*` — page routes. Example: `src/pages/index.astro`, `src/pages/blackjack/*.astro`.
  - `src/layouts/BaseLayout.astro` — main page shell (header, Sidebar import, footer). Use this for site-wide layout changes.
  - `src/components/` — shared components (e.g., `Sidebar.astro`).
  - `src/pages/game/` — a small interactive React/TS feature:
    - `src/pages/game/index.tsx` (React component exported as `BlackjackGame`). It uses `@astrojs/react` for client-side interactivity.
    - `src/pages/game/hooks/hooks.ts` — simple React hook `useGameEngine` providing deal/hit/stand.
  - `src/styles/` — global styling tokens. `BaseLayout.astro` links `/styles/global.css`.

Patterns & conventions
- Astro + React interop: interactive UI lives under `src/pages` as React components imported/used inside Astro pages. Look for `import React` and `@astrojs/react` usage.
- File types: prefer editing `.astro` for pages/layouts and `.ts/.tsx` for interactive logic. Keep TypeScript strictness in mind (tsconfig extends `astro/tsconfigs/strict`).
- Styling: small site styles are referenced at `/styles/global.css` and `src/styles/tokens.css`. Inline styles are used in several components — follow existing simple, inline style approach for quick patches.
- Small scope for JS/TS logic: game logic is intentionally minimal (emoji cards). When changing behavior, update both the hook (`hooks.ts`) and the consuming component (`index.tsx`) to keep API consistent (`dealHand` vs `onDeal` naming mismatch — check local names).

Developer workflows (what to run)
- Install: `pnpm install` (project uses pnpm in lockfile) or `npm install` if pnpm unavailable.
- Dev server: `pnpm run dev` — opens Astro dev server (default port 4321).
- Build & preview: `pnpm run build` then `pnpm run preview` to test production output.

Common tasks & examples
- Add a new documentation page: create `src/pages/<name>.astro`, wrap content in `<BaseLayout title="...">...</BaseLayout>`.
- Update layout or sidebar: edit `src/layouts/BaseLayout.astro` and `src/components/Sidebar.astro`.
- Modify game logic: the hook is at `src/pages/game/hooks/hooks.ts`. If adding new exported functions, update `src/pages/game/index.tsx` to consume them.

Notes about discovered inconsistencies (fix-first suggestions)
- In `src/pages/game/index.ts` the component imports `useGameEngine` and calls `dealHand()` but the hook `hooks.ts` exposes `onDeal`, `onHit`, `onStand` and state fields — there is no `dealHand` export. When updating behavior, align naming: either export `dealHand` from the hook or change the component to call `onDeal`.

Search/replace hints
- When changing routes, remember Astro route file path === URL. Use `src/pages/blackjack/index.astro` → `/blackjack`.

What to avoid
- Don't introduce server-only Node APIs in `.astro` frontmatter without checking Astro docs. Keep client-side React code in `.tsx` or inside `client:load` / `client:idle` islands if needed.

If you're unsure
- Look at `package.json` scripts and `tsconfig.json` for build and typing rules.
- Run the dev server locally to validate layout/shapes before opening PR.

Files referenced in these notes (examples):
- `package.json`, `tsconfig.json`, `astro.config.mjs`
- `src/layouts/BaseLayout.astro`, `src/components/Sidebar.astro`
- `src/pages/index.astro`, `src/pages/blackjack/index.astro`, `src/pages/game/index.ts`, `src/pages/game/hooks/hooks.ts`

If anything above is unclear or you want the instructions shortened/expanded, tell me what to change and I'll iterate.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rajahwu)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/rajahwu)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
