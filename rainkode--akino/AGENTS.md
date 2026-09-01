# Copilot Instructions — Akino Studio

## Project Overview

Marketing site for **Akino Studio**, a video production agency. React 19 + Vite 7 with `react-router-dom` v7 for routing. The homepage renders sequential sections inside a Lenis smooth-scroll provider; separate routes serve `/blog`, `/blog/:slug`, and `/privacy-policy`. Blog content is sourced from **Sanity CMS** with static fallback data in `src/components/blogData.js`. No tests, no linter, no CI.

## Architecture & Routing

- **Router**: `BrowserRouter` wraps `<App />` in `src/main.jsx`. Routes defined in `src/App.jsx`:
  - `/` → `<HomePage />` (Lenis-wrapped section sequence)
  - `/privacy-policy` → `<PrivacyPolicy />`
  - `/blog` → `<Blog />` (lists posts from Sanity, falls back to `blogData.js`)
  - `/blog/:slug` → `<BlogPost />` (single post, Sanity → static fallback)
- **Homepage section order**: `Navbar → Hero → WhatWeDo → ProjectShowcase → Process → Testimonials → FAQ → Contact → Footer`
- **Smooth scrolling**: Lenis (`lenis/react` → `<ReactLenis root>`) wraps `HomePage` only. Navigation uses `document.getElementById(id)?.scrollIntoView({ behavior: 'smooth' })`.

## Sanity CMS Integration

- **Client**: `src/sanityClient.js` — project `15a0z2p7`, dataset `production`, CDN-enabled read-only.
- **Studio**: Separate project in `studio/` with its own `package.json`. Schema at `studio/schemas/post.js`.
- **Blog data flow**: `Blog.jsx` fetches GROQ query → if empty/error, renders `blogData.js` static posts. `BlogPost.jsx` renders Sanity rich text via `@portabletext/react`, or static HTML content via `dangerouslySetInnerHTML`.
- **Schema reference** also in `src/sanitySchema.js` (documentation copy, not used at runtime).

## Forms & External Services

- **Contact form** (`Contact.jsx`): Posts to **Web3Forms** API (`https://api.web3forms.com/submit`). Includes honeypot spam field.
- **Newsletter** (`Footer.jsx`): Also Web3Forms, separate access key. Both use inline status states (`idle | sending | success | error`).

## CSS & Styling

- **Co-located CSS**: Each component has a paired `.css` file (e.g., `Hero.jsx` + `Hero.css`). No CSS modules, no Tailwind.
- **BEM-like classes**: `.hero-stack--primary`, `.faq-item--active`, `.expand-card-overlay`.
- **Design tokens** in `src/index.css` `:root`: `--primary` (#03a6af), `--secondary` (#fd5863), `--dark`, `--background`, `--radius`, `--transition`. Always use these — never hardcode colors.
- **Fonts**: Inter (body) loaded via Google Fonts in `index.html`.

## Key Conventions

- **Component pattern**: Functional arrow-function components, default export. Data arrays declared as `const` above the component — never inline in JSX.
- **Motion library** (package: `motion`):
  - Animated JSX / `AnimatePresence` / `useInView`: import from **`motion/react`** — e.g., `import { motion, AnimatePresence } from 'motion/react'`
  - Imperative scroll animations: import from **`motion`** — e.g., `import { animate, scroll } from 'motion'` (see `WhatWeDo.jsx`)
  - **Never** import from `framer-motion`
- **Icons**: `lucide-react` for standard icons; inline SVGs for custom icons (hamburger, close, arrows).
- **Section tags**: `<span className="section-tag">&lt; Label &gt;</span>` with HTML entities.
- **Scroll-driven state**: `Hero`, `Process`, `ProjectShowcase` compute active index from scroll via `useRef` + `window.addEventListener('scroll', ..., { passive: true })`. Use this pattern, not IntersectionObserver.
- **Video cards**: `ProjectShowcase` supports two types — `LocalVideoCard` (hover-to-play `.webm` from `public/videos/`) and `YouTubeVideoCard` (hover-to-embed iframe). Thumbnails: `.jpg` for local, YouTube `mqdefault` for remote.
- **Sub-page pattern** (`Blog.jsx`, `PrivacyPolicy.jsx`): Each calls `window.scrollTo(0, 0)` and sets `document.title` in `useEffect`, restoring the default title on cleanup. Uses `<Link to="/">` for back navigation.

## Unused / Orphaned Code

Do not import for new work:
- `components/uilayouts/accordion.jsx` + `lib/utils.js` — root-level legacy (Tailwind Merge `cn()`), unused by `src/`.
- `FeaturedWork.jsx` — imported in `App.jsx` but **not rendered**. Dormant.
- `EveryFrame.jsx` / `EveryFrame.css` — not imported anywhere.

## Development

```bash
npm run dev      # Vite dev server (HMR)
npm run build    # Production build
npm run preview  # Preview production build
```

Sanity Studio (separate project): `cd studio && npx sanity dev`

No test runner or linter is configured.

## Adding a New Homepage Section

1. Create `src/components/SectionName.jsx` + `src/components/SectionName.css`.
2. Import and place `<SectionName />` in the `HomePage` function in `App.jsx` at the correct position.
3. Add `id` attribute to the section root for scroll navigation.
4. To appear in nav: add entry to `navItems` in `Navbar.jsx` (`{ label, target }` mapping to section ID).
5. Use CSS custom properties from `src/index.css`.

## Adding a New Route

1. Create component + CSS in `src/components/`.
2. Add `<Route path="..." element={<Component />} />` in `App.jsx` inside `<Routes>`.
3. Follow the sub-page pattern: `window.scrollTo(0, 0)` + `document.title` in `useEffect`, `<Link to="/">` back nav.

---
> Source: [RainKode/Akino](https://github.com/RainKode/Akino) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-01 -->
