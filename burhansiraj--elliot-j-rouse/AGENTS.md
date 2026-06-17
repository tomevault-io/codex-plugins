# Copilot Instructions for Elliott Rouse Portfolio

## Build & Development Commands

- **Development server:** `npm run dev` - Starts Next.js dev server at http://localhost:3000
- **Production build:** `npm run build` - Compiles TypeScript and optimizes for Vercel deployment
- **Production start:** `npm run start` - Runs production-optimized server
- **Linting:** `npm run lint` - ESLint with max-warnings=0 (must pass cleanly)
- **Install dependencies:** `npm install`

No test suite exists. The project uses ESLint (Next.js core and TypeScript configs) with strict linting.

## Architecture Overview

### Technology Stack
- **Framework:** Next.js (App Router) with TypeScript
- **Styling:** Tailwind CSS with custom theme colors (primary, ink, panel, paper, soft, copper)
- **Animation:** GSAP with ScrollTrigger for scroll-triggered reveals
- **Smooth scrolling:** Lenis for physics-based scroll behavior
- **UI Icons:** Lucide React
- **Image optimization:** Next.js Image component with responsive sizing and GIF support
- **Media hosting:** Static assets in `/public/media` (pre-normalized)

### Project Structure
- **`app/`** - Next.js App Router pages and layout
- **`components/`** - Reusable React components (Hero, ProjectCard, Reveal, etc.)
- **`content/portfolio.ts`** - Centralized data: projects, publications, expertise, honors, navigation
- **`hooks/`** - Custom React hooks (usePrefersReducedMotion)
- **`public/media/`** - Optimized media files (images, videos)
- **`app/globals.css`** - Global CSS with Tailwind + custom theme variables

### Page Composition Pattern
The main portfolio is a single-page app with semantic sections:
1. Hero (video background with GSAP animations)
2. Work section (project grid with Reveal animations)
3. Expertise section (3-column grid on large screens)
4. Publications (sticky sidebar + scrolling list)
5. Honors (4-column grid)
6. About section (image + biography)
7. Lab section (full-width image with overlay text)
8. Contact section

Each section uses the **Reveal component** for staggered scroll-triggered animations.

## Key Conventions

### Animation & Motion
- **Reveal wrapper:** Wrap content that needs scroll-triggered fade/slide animations in `<Reveal delay={index * 0.05}>`. Respects prefers-reduced-motion.
- **GSAP patterns:** Use `gsap.fromTo()` with `duration`, `ease: "power3.out"`, and `stagger` for sequences
- **ScrollTrigger:** Register plugin, set `trigger` to element, use `start: "top 84%"` and `once: true`
- **Accessibility:** Always respect `prefers-reduced-motion` preference; use `data-hero-reveal` attributes for manual GSAP selectors

### Styling & Tailwind
- **Custom colors:** Use semantic class names: `bg-ink` (dark bg), `bg-paper` (light text), `text-primary` (teal accent), `bg-panel` (slightly lighter dark), `text-soft` (muted light), `text-copper` (warm accent)
- **Layout utilities:** Use Tailwind's responsive prefixes (`md:`, `lg:`, `xl:`) heavily; mobile-first
- **Component containers:** Use `.section-shell` class for consistent padding and max-width
- **Grid patterns:** Use `md:grid-cols-2 lg:grid-cols-3` for responsive grids
- **Ring styles:** Use `.primary-ring` utility class for button/card focus states
- **Transitions:** Use `transition duration-700` for hover effects

### Data & Content
- **Single source of truth:** All content (projects, publications, expertise, honors, nav items) lives in `content/portfolio.ts`
- **Data structure:** Export named arrays (projects, publications, expertise, honors, navItems, heroLabels)
- **Typed exports:** Use TypeScript interfaces (Project, Publication, etc.) for type safety
- **External links:** Always use `target="_blank" rel="noreferrer"` for external URLs

### Image Optimization
- **Priority loading:** Set `priority={true}` for first 2 images in viewport (ProjectCard uses `index < 2`)
- **GIF handling:** Use `unoptimized={true}` for animated GIFs to prevent optimization artifacts
- **Responsive sizing:** Use `sizes` prop with breakpoints: `"(min-width: 1024px) 50vw, 100vw"`
- **Next.js Image:** Always use `next/image` Image component, not `<img>` tag

### Metadata & SEO
- **Page metadata:** Define in `app/layout.tsx` using Next.js Metadata API
- **Viewport theme:** Set `themeColor` and `colorScheme` in layout.tsx
- **Image alt text:** Use semantic, descriptive alt text for all images
- **Semantic HTML:** Use `<section>` with id anchors, `<article>` for card components

### Navigation & Links
- **Hash scrolling:** Navigation items link via hash ids (e.g., `href="#work"`)
- **Scroll padding:** Root element has `scroll-padding-top: 88px` to account for fixed nav
- **Overlay menu:** OverlayMenu component manages mobile/desktop nav state

### Component Patterns
- **"use client":** Mark interactive components (animations, event listeners) with "use client"
- **Props typing:** Use explicit TypeScript types for component props (see Reveal, ProjectCard examples)
- **Ref management:** Use `useRef` for DOM references in animation components; always clean up refs in useEffect returns
- **Hooks:** Custom hooks live in `hooks/` directory and use "use" prefix

### Build & Deployment
- **Type checking:** TypeScript in strict mode (noEmit: true)
- **Module resolution:** Uses bundler strategy with path aliases (`@/*`)
- **Vercel deployment:** Supports Vercel natively via `next.config.ts`; uses `.vercelignore` for exclusions
- **Linting:** ESLint config extends Next.js core + TypeScript; must pass without warnings

---
> Source: [BurhanSiraj/elliot-j-rouse](https://github.com/BurhanSiraj/elliot-j-rouse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-17 -->
