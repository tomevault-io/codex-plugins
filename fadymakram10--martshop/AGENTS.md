## Quick orientation for AI coding agents

This Next.js project uses the App Router (the `src/app` folder) with TypeScript and TailwindCSS. Keep edits minimal and follow existing patterns in `src/components` and `src/app`.

- Entry points:
  - `src/app/layout.tsx` — top-level layout, imports `@/components/navbar/page` and global CSS (`globals.css`).
  - `src/app/page.tsx` — main landing page.

- Build & dev commands (from `package.json`):
  - Install deps: `npm install` (or `pnpm/yarn` if preferred)
  - Dev server: `npm run dev` → Next dev at http://localhost:3000
  - Build: `npm run build` and run: `npm run start`
  - Lint: `npm run lint` (project uses `eslint`)

- Important conventions and patterns
  - Path alias: `@/*` → `src/*` (see `tsconfig.json`). Use `@/` imports throughout (e.g. `@/lib/utils`, `@/components/ui/button`).
  - UI components live in `src/components/ui/*`. Follow existing component patterns (props, exported subcomponents). Example: `NavigationMenu` in `src/components/ui/navigation-menu.tsx` uses Radix primitives and `cn()` from `src/lib/utils.ts`.
  - Small utility: `src/lib/utils.ts` exports `cn(...)` that wraps `clsx` + `twMerge` — use it for conditional classNames.
  - Types: domain types live in `src/interfaces/*.ts` (product, brand, category, subcategory).
  - Styling: Tailwind v4 + utility classes. Many components use `class-variance-authority` for variant-driven styles (see `button.tsx`, `card.tsx`).

- Integration and infra specifics
  - `next.config.ts` includes an `images.remotePatterns` entry for `ecommerce.routemisr.com` and path `/Route-Academy-*/*`. If you add remote images, ensure they match that config or update it.
  - Fonts loaded with `next/font/google` in `layout.tsx`. Keep font variables (`--font-geist-sans`, `--font-geist-mono`) applied on <body>.

- Component authoring rules (observable from codebase)
  - Prefer small, focused components in `src/components/ui`. Export named subcomponents where present (see `navigation-menu.tsx` which exports Trigger, Content, Link, etc.).
  - Use `data-*` attributes used in Radix primitives (e.g., `data-slot` or `data-state`) — preserve them when modifying markup.
  - Keep semantic structure: layout imports `Navbar` as a component (not a route). Some components are under `components/*/page.tsx` — treat them as reusable page-like components.

- What to avoid changing without consent
  - Global CSS variables and Tailwind config (styles live in `src/app/globals.css`).
  - `next.config.ts` image host rules and TypeScript path mappings.

- Next steps / questions for the user
If anything here is unclear or you want more details (examples of tests, API fetch patterns, or where data is fetched), tell me which area and I will expand or merge content from other files.

---
> Source: [Fadymakram10/martshop](https://github.com/Fadymakram10/martshop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-12 -->
