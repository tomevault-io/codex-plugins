<!-- .github/copilot-instructions.md -->

# Copilot / AI assistant instructions — portfolio

Purpose

- Help AI contributors get productive quickly in this Next.js + TypeScript portfolio repo.
- Focus on observable, actionable patterns and files an automated agent will modify.

Quick facts

- Framework: Next.js 14 (App Router). See `app/` for routes and layouts. Server and client components are used.
- Language: TypeScript. Project-wide types live in `config/portfolio.ts` which is the single source of truth for content.
- Styling: Tailwind CSS (see `tailwind.config.js`) + `app/globals.css` for CSS variables and dark/light themes.
- Image handling: Uses Next.js `Image` + `next.config.js` image settings (AVIF/WebP, remotePatterns). Prefer `public/images/` for local assets.

Where to look first (high-value files)

- `app/layout.tsx` — root layout, theme provider, metadata wiring (reads `config/portfolio.ts`).
- `app/page.tsx` — home page server component; calls `lib/fetch` helpers like `lib/github.ts` to load projects.
- `config/portfolio.ts` — primary content/config. Edit this to change personal info, skills, projects, SEO, etc.
- `lib/github.ts` — GitHub fetch helpers; used by `app/page.tsx` and scripts. See `scripts/fetch-github-projects.ts` for offline fetch workflow.
- `components/` — UI components and well-scoped subfolders (`sections`, `ui`, `navigation`). Follow existing component patterns (props, skeletons, small focused components).
- `app/api/resume/route.ts` — server route that returns the resume (used by `config.portfolio.personal.social.resumeLink`).
- `docs/` — implementation notes (image & performance best-practices). Check before changing images or animations.

Developer workflows & commands

- Install: `npm install`
- Dev: `npm run dev` (runs `next dev`) — use this to validate UI/hot-reload changes.
- Build: `npm run build` then `npm start` for production verification.
- Lint: `npm run lint` (Next lint integration).
- Fetch projects (helper script): `npm run fetch-projects` runs `scripts/fetch-github-projects.ts` via `ts-node` to populate or test GitHub project fetching.

Project-specific conventions (do not assume defaults)

- Single source of site content: update `config/portfolio.ts` for any change to personal info, projects, skills, SEO, images paths, etc. Many components import this file directly.
- Server-first data fetching: pages like `app/page.tsx` are async server components and call `lib` functions (e.g., `fetchGitHubProjects`). Preserve server/component boundaries when moving code.
- Images: Prefer Next.js `Image` and local assets in `public/images/`. If you add external sources, update `next.config.js` `remotePatterns` accordingly.
- Suspense/loading: Project uses Suspense boundaries and loading skeletons (see `app/loading.tsx` and components like `home-with-loading.tsx`); when adding async UI, provide fallbacks.
- Theming: `ThemeProvider` is used in `app/layout.tsx` (see `components/theme-provider.tsx`); theme changes should preserve CSS variable usage in `globals.css`.
- Absolute imports: code uses `@/` path alias (check `tsconfig.json`); prefer these imports to relative path climbing.

Integration points & environment

- Environment variables: `NEXT_PUBLIC_SITE_URL`, `NEXT_PUBLIC_GOOGLE_ANALYTICS` — used in `next.config.js` and `app/layout.tsx`.
- Third-party: GitHub (via `@octokit/graphql`) for dynamic projects; check `lib/github.ts` and script in `scripts/`.
- Analytics: Google Analytics optional integration via `@next/third-parties/google` in `app/layout.tsx`.

Examples (copy-paste friendly)

- Update site bio/SEO: edit `config/portfolio.ts` -> `personal` and `seo` sections. `app/layout.tsx` will pick up changes automatically on reload.
- Add a project (quick):
  1. Add an entry to `config/portfolio.ts` `projects` array. Include `id`, `title`, `description`, `image` (in `public/images/projects/` or full HTTPS URL), `technologies`.
  2. If using external images, add an allowed `remotePattern` in `next.config.js`.
  3. Run `npm run dev` and confirm it appears on `/`.
- Re-generate GitHub projects data: `npm run fetch-projects` — used to test the GitHub fetch logic outside the runtime.

Checks and cautions for automated changes

- When changing images: ensure `width`/`height` (or layout appropriate props) are set on `Image`, and confirm `next.config.js` supports the domain.
- When adding third-party libraries: check bundle impact (see `docs/PERFORMANCE_OPTIMIZATIONS.md`) and aim to keep performance budgets.
- Avoid modifying `app/layout.tsx` metadata without updating `config/portfolio.ts` — metadata is generated from the config file.
- Keep server-only code (secrets, long-running fetch) out of client components. Use server routes (`app/api/`) or server components.

What I (AI) should do automatically

- Make small content edits in `config/portfolio.ts` when user asks to update name, bio, skills, or projects.
- Add small UI components under `components/` following existing naming and styling patterns. Use `ui/` primitives when available.
- For image changes, place assets in `public/images/` and update `config/portfolio.ts`; update `next.config.js` if external domains are required.

If you're unsure where to edit

- Start at `config/portfolio.ts` for content changes.
- For UI layout or visual work, inspect `app/layout.tsx` and `components/sections/*`.
- For data-fetching behavior, inspect `lib/*` and `app/page.tsx`.

If anything above is unclear, tell me which part you'd like expanded (build scripts, data flow, or component patterns) and I'll iterate.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ayushjaipuriyar)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/ayushjaipuriyar)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
