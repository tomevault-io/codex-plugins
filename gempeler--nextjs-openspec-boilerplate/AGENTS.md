# mission_stratos_where AI Guide

- **Stack**: Next.js 16 App Router in `src/app` with React 19; no Pages router present.
- **Entry layout**: `src/app/layout.js` wires next/font Geist families and exports `metadata`; reuse these when adding routes to inherit fonts and metadata.
- **Global styling**: `src/app/globals.css` imports Tailwind CSS 4 via `@import "tailwindcss"` and defines theme tokens with CSS custom properties; extend theme in this file instead of a Tailwind config file.
- **Dark mode**: prefer CSS variables already defined (`--background`, `--foreground`) and respect the `prefers-color-scheme` overrides when adding styles.
- **Routing**: Add new routes by creating folders under `src/app/<route>/page.js`; co-locate route-specific layout/components inside the route folder.
- **Images**: Use `next/image` as shown in `src/app/page.js` for automatic optimization; remember to set `priority` cautiously.
- **Path aliases**: Import shared modules with the `@/` alias (configured in `jsconfig.json`) to avoid relative paths.
- **Static assets**: Place files under `public/` and reference them with root-relative URLs (`/asset.svg`).
- **Fonts**: Stick with `next/font/google` for additional fonts to keep them server-side bundled.
- **Config**: `next.config.mjs` is empty by default—only add keys you truly need to keep the build minimal.
- **Data fetching**: No client/server components defined yet; default `page.js` is a Server Component. Introduce `"use client"` at the top only when using hooks or browser-only APIs.
- **Styling utilities**: Tailwind classes (e.g., `bg-zinc-50`, `dark:text-zinc-50`) are already in use; avoid manual CSS unless necessary.
- **State management**: None added yet—favor React server components and lift state carefully if you introduce client components.
- **Build tooling**: Use npm scripts; run `npm run dev` for local dev, `npm run build` before deployment, `npm run start` for production preview.
- **Linting**: `npm run lint` runs Biome with `next`/`react` recommended rules and allows unknown `@` at-rules for Tailwind; fix issues before committing.
- **Formatting**: `npm run format` uses Biome's formatter with 2-space indentation; do not mix with other formatters (Prettier, eslint --fix).
- **CI expectations**: No CI defined yet—assume Biome and Next build must succeed locally.
- **Dependencies**: Tailwind 4 via `@tailwindcss/postcss`; no separate config file, so rely on PostCSS plugin defined in `postcss.config.mjs`.
- **Import hygiene**: Biome auto-organizes imports; keep imports minimal and rely on named exports.
- **Environment**: No `.env` usage currently; add new secrets via Next.js standard `.env.local` if needed and document keys in commit.
- **Testing**: No test harness configured; if you add one, wire npm scripts so others can reuse.
- **TypeScript**: Project currently JavaScript; if adding TypeScript files, ensure `jsconfig` is upgraded to `tsconfig` and update instructions.
- **Performance**: Default Next static assets; prefer server components to keep bundle small unless interactivity is required.
- **Accessibility**: Use semantic HTML elements like the existing `main` in `page.js`; keep hitting `aria` requirements when adding interactive components.
- **Deployment**: README points to Vercel defaults; ensure `npm run build` passes before pushing deployment changes.
- **OpenSpec**: Never implement, revert, or apply spec-driven changes unless the user explicitly issues an `/openspec:apply` command for that change.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gempeler)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/gempeler)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
