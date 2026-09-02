## Copilot / AI agent instructions for the lyannes repo

Purpose: short, actionable guidance so an AI coding agent can be immediately productive in this Next.js app.

Key facts (big picture)

- This is a Next.js (app directory) TypeScript project (Next v16.1.1, React 19.2.3). Primary UI lives under `app/`.
- The app uses the app-router conventions: top-level layout in `app/layout.tsx` and pages under `app/` (e.g. `app/page.tsx`).
- Styling uses Tailwind + PostCSS. PostCSS plugin is configured in `postcss.config.mjs` (uses `@tailwindcss/postcss`). There is no `tailwind.config.*` file in the repo root currently.

Where to look first (high-value files)

- `app/layout.tsx` — global layout, font variables, where server/client boundaries often start.
- `app/page.tsx` — homepage example and component/classname patterns.
- `globals.css` (under `app/`) — global styles and Tailwind base imports (used by layout).
- `package.json` — scripts and dependency versions (dev/start/build/lint).
- `postcss.config.mjs` — PostCSS plugin list (Tailwind entry point).
- `next.config.ts` and `tsconfig.json` — Next/TS settings and path alias `@/*`.
- `public/` — static assets (images used by `next/image`).

Project-specific conventions & examples

- App dir: treat files/folders under `app/` as routes. To add `/about`, create `app/about/page.tsx`.
- Layout and font vars: `app/layout.tsx` uses next/font to set CSS variables like `--font-geist-sans`. When adding components that use the global font, reference these variables via className or CSS custom props.
- Styling: components use Tailwind utility classes inline in JSX (see `app/page.tsx`). Keep to this pattern rather than creating many bespoke CSS modules unless necessary.
- Images: use `next/image` with files from `public/` (see `app/page.tsx` usage of `/next.svg`).
- TypeScript: `tsconfig.json` sets `strict: true` and a path alias `@/*` → `./*`. Use that alias for repo-local imports when helpful.

> Example quick edits

- To change the homepage text: edit `app/page.tsx` (top-level text lives in the <h1>).
- To add a new route: add `app/<route>/page.tsx` and export a React component as default.

Scripts & developer workflows

- Development: `npm run dev` → runs `next dev` (local server at http://localhost:3000).
- Build: `npm run build` → `next build`.
- Start (production): `npm run start` → `next start`.
- Lint: `npm run lint` runs `eslint` (package.json defines script as `eslint`). The project includes `eslint-config-next`. If running lint manually, pass target paths (e.g., `npx eslint . --ext .ts,.tsx`) as needed.

Integration points & deployment

- Vercel is the natural host (standard Next support). `app/` routing, image handling, and fonts work out-of-the-box on Vercel.
- No API routes or backend services are present in the repo (no `pages/api` or `app/api` entries). If adding server endpoints, use `app/api/*` or a separate backend integration.

What the agent should avoid or check first

- There is no `tailwind.config.js` — if you add Tailwind utility customization, add a `tailwind.config.{js,cjs,mjs}` at repo root and ensure `postcss.config.mjs` is consistent.
- Confirm `eslint` configuration before running `npm run lint` in CI; the script is bare `eslint` and may require paths/args depending on local CLI.

Small checklist for code changes

1. Run `npm run dev` locally and verify UI change in `app/`.
2. If types change, confirm `tsc`/Next build (`npm run build`) passes—`tsconfig.json` uses `strict: true`.
3. If adding Tailwind config, include it in PR and ensure PostCSS plugin remains present in `postcss.config.mjs`.

If anything here is unclear or you want additional conventions (commit message style, testing patterns, CI), tell me which area to expand and I will update this file.

---

## Lyanne's Brand & Design Guidelines

Lyanne's is a premium SME reselling luxury beauty, skincare, makeup, and home luxury products (sheets, blankets, covers). The landing page must convey elegance, minimalism, and a high-end boutique feel—heavily inspired by Zara's web aesthetic.

### Visual Identity

- **Typography**: Thin, elegant fonts (font-weight 100–300). Use CSS `font-weight: 200` or Tailwind `font-extralight`/`font-light` as defaults. Headlines may go up to `font-normal` (400) sparingly.
- **Color palette**: Predominantly white/off-white backgrounds, deep black (#000) text and accents. Avoid bright colors; keep to monochrome with occasional muted tones (cream, blush, soft grey).
- **Buttons**: Transparent background, thin black border (`border border-black`), rectangular shape, thin font. Hover: subtle fill or border thickening.
- **Images**: Large, full-bleed, high-quality product/lifestyle imagery. Prefer aspect ratios that fill sections.

### Page Structure (Single-Page Layout)

1. **Top Contact Strip** – Very thin bar above navbar with phone number + WhatsApp link (icon + text).
2. **Navbar** – Sticky, minimal. Anchor links to sections (`#hero`, `#brands`, `#products`, `#contact`). Logo left, links right. Thin font.
3. **Hero Section** – Full-viewport background image, minimal overlay text (brand tagline), single CTA button.
4. **Brands Carousel** – Horizontal infinite-scroll carousel of partner logos (LV, Hermès, Nivea, Zara, etc.).
5. **Product/Category Sections** – Parallax image backgrounds with floating text blocks. Animate on scroll (fade-in, slide-up).
6. **Footer / Contact** – Minimal footer with social links and contact info.
7. **Quick Action FAB** – Floating button (bottom-right) for WhatsApp or cart quick-access.

### Animations & Effects

- **Parallax**: Use CSS `background-attachment: fixed` or a lightweight JS parallax library for depth.
- **Scroll animations**: Fade-in / slide-up on viewport entry. Use Intersection Observer or a library like `framer-motion`.
- **Carousel**: Smooth infinite loop for brand logos; consider `embla-carousel` or CSS keyframe animation.

### Component Conventions

- Place reusable components in `app/components/` (e.g., `Navbar.tsx`, `Hero.tsx`, `BrandCarousel.tsx`).
- Keep each component focused; prefer composition over monolithic files.
- Use Tailwind utilities; avoid custom CSS unless necessary for animations.

### Assets

- Store images in `public/images/` (hero backgrounds, product shots, brand logos).
- Use `next/image` for optimization; set `priority` on hero image.

### Example Button Markup

```tsx
<button className="border border-black bg-transparent px-6 py-2 text-sm font-extralight tracking-widest uppercase hover:bg-black hover:text-white transition-colors">
  Shop Now
</button>
```

### Example Thin Text

```tsx
<h1 className="text-4xl font-extralight tracking-wide">Luxury Redefined</h1>
```

---
> Source: [Derngineer/kuzman-](https://github.com/Derngineer/kuzman-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-02 -->
