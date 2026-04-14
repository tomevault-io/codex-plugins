# Copilot Instructions — Dinesh Wayaman Portfolio

## Who is Dinesh Wayaman?

Dinesh Wayaman is a Flutter & React Native specialist and full-stack freelancer based in Colombo, Sri Lanka. He has 5+ years of freelance experience, shipped 50+ cross-platform mobile apps, and served clients across 10+ countries. He writes about mobile engineering at [speedicode.com](https://speedicode.com).

**Contact:** hello@dineshwayaman.com  
**Location:** Colombo, Sri Lanka  
**GitHub:** https://github.com/dineshwayaman  
**LinkedIn:** https://linkedin.com/in/dineshwayaman

---

## Portfolio Purpose

A dark-themed, mobile-first personal portfolio that showcases Dinesh's work, skills, and freelance journey. It is statically rendered, SEO-optimised, and performance-focused (target: < 150 kB JS first load, Lighthouse ≥ 95).

---

## Tech Stack (version-pinned)

| Package | Version |
|---------|---------|
| Next.js | 16.x (App Router) |
| React | 19.x |
| TypeScript | 5.x (strict) |
| Tailwind CSS | 4.x (CSS-first, no config file) |
| PostCSS | via `@tailwindcss/postcss` |

---

## Key Architecture Patterns

### 1. Server Components first
All sections (`Hero`, `About`, `TechStack`, `Projects`, `Timeline`, `Blog`) are Server Components unless interactivity is required. Only add `"use client"` when using React hooks (e.g., `useState`, `useEffect`, `useRef`).

Client components in this project:
- `components/layout/Header.tsx` — scroll detection + IntersectionObserver
- `components/sections/About.tsx` — scroll-reveal animation
- `components/sections/Contact.tsx` — clipboard copy

### 2. Data centralisation
**ALL portfolio copy lives in `lib/data.ts`.** Never hardcode text in components. Import from `lib/data.ts`.

```ts
import { profile, stats, skills, projects, timeline, socialLinks } from "@/lib/data";
```

### 3. Tailwind v4 CSS-first syntax
- **No `tailwind.config.ts`** — Tailwind v4 is configured in CSS only.
- Design tokens are defined in `app/globals.css` under `@theme inline { … }`.
- Import Tailwind with `@import "tailwindcss";` (not `@tailwind base;` etc.).
- Custom theme colours: `bg-background`, `bg-surface`, `text-accent`, `text-muted`, `border-border`, `text-accent2`.

### 4. Path aliases
Use `@/` alias for all internal imports (configured in `tsconfig.json`).

```ts
import { profile } from "@/lib/data";             // ✅
import { profile } from "../../lib/data";          // ❌
```

---

## Design System

| Token | CSS Var | Value | Use |
|-------|---------|-------|-----|
| Background | `--background` | `#09090B` | Page base |
| Surface | `--surface` | `#18181B` | Cards |
| Border | `--border` | `#3F3F46` | Dividers |
| Accent | `--accent` | `#06B6D4` | CTA, highlights |
| Accent 2 | `--accent-2` | `#8B5CF6` | Badges, tags |
| Text | `--text` | `#FAFAFA` | Headings |
| Muted | `--text-muted` | `#A1A1AA` | Body copy |

Font: Geist Sans via `--font-sans`.

### Animations (CSS only)
Defined in `globals.css`:
- `.animate-fade-up` — entrance animation
- `.animate-float` — floating phone mockup
- `.animate-shimmer` — gradient text shimmer
- `.animate-gradient-shift` — hero background
- `.animate-pulse-glow` — phone frame glow
- `.reveal` / `.reveal.revealed` — scroll-triggered

---

## File Structure

```
app/
  layout.tsx          ← root metadata + JSON-LD (Person + WebSite schema)
  page.tsx            ← composes all sections (Server Component)
  sitemap.ts          ← Next.js sitemap route
  robots.ts           ← Next.js robots.txt route
  globals.css         ← Tailwind v4 @theme tokens + keyframes

components/
  sections/           ← Hero, About, TechStack, Projects, Timeline, Blog, Contact
  ui/                 ← Button, Badge, SectionTitle, Card
  layout/             ← Header (client), Footer
  icons/              ← GitHubIcon, LinkedInIcon, GlobeIcon (SVG components)

lib/
  data.ts             ← ALL portfolio content (single source of truth)
  metadata.ts         ← buildMetadata() helper

types/
  index.ts            ← Profile, Stat, Skill, SkillCategory, Project, TimelineEntry, SocialLink
```

---

## Accessibility Requirements

- Semantic HTML: `<main>`, `<section>`, `<article>`, `<nav>`, `<header>`, `<footer>`
- Exactly one `<h1>` on the page (in `Hero.tsx`)
- Descending heading hierarchy: `<h1>` → `<h2>` (section titles) → `<h3>` (card titles)
- `aria-label` on all `<section>` elements
- `aria-hidden="true"` on decorative SVGs and elements
- `alt` on every `<img>` / `next/image`
- Focus-visible ring on all interactive elements
- Sufficient colour contrast (WCAG AA minimum)

---

## Performance Rules

- Prefer Server Components; avoid `"use client"` unless necessary
- No Framer Motion (CSS animations + IntersectionObserver instead)
- No heavy icon libraries (inline SVG only)
- `next/image` for all raster images; `loading="lazy"` on non-hero images
- No contact form backend — mailto link + social links only
- Target: < 150 kB JS first load; Lighthouse Performance ≥ 95

---

## SEO

- `buildMetadata()` from `lib/metadata.ts` generates full OG/Twitter metadata.
- JSON-LD `Person` + `WebSite` schema injected in `app/layout.tsx`.
- Sitemap at `/sitemap.xml` and robots at `/robots.txt` via Next.js file-based routes.
- Canonical URL: `https://dineshwayaman.com`

---

## Common Gotchas

1. **Tailwind v4 tokens** — Use CSS variable‐based class names like `bg-background`, not hardcoded `bg-zinc-950`. The mapping is in `@theme inline {}` in `globals.css`.
2. **Server vs Client** — Adding `useState`/`useEffect` requires `"use client"` at the top of the file.
3. **`@keyframes` in Tailwind v4** — Define keyframes directly in `globals.css`, not in a config file.
4. **`next/font`** — Fonts are loaded in `app/layout.tsx` and exposed as CSS variables (`--font-geist-sans`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DineshWayaman) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
