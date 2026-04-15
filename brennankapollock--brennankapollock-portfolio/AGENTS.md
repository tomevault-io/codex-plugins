# Copilot Instructions

## Project Overview
Personal portfolio + blog platform for Brennan K.A. Pollock — Venice Beach-based engineer and artist. The site embodies "minimal yet expressive" design inspired by museum systems (Stedelijk), brutalist web aesthetics, and developer-first editorial platforms (Stripe Dev Blog).

**Stack**: Next.js 15 (App Router), React 19, Tailwind CSS v4 (via PostCSS), JavaScript only (no TypeScript), Biome for linting/formatting.

## Architecture

### Styling System (CRITICAL)
- **Tailwind v4 via PostCSS**: NO `tailwind.config.*` file — theme tokens declared inline in `src/app/globals.css` using `@theme` blocks
- **CSS class naming**: Uses `.museum-*` and `.words-*` BEM-style prefixes for custom components (see globals.css)
- **NO @apply directives**: Write vanilla CSS in globals.css, use Tailwind utility classes in JSX
- **Font system**: 
  - Display: Printvetica (local, `--font-display`)
  - Sans: Geist Sans (`--font-geist-sans`)
  - Mono: Departure Mono (local, primary for articles/blog) OR Geist Mono (fallback)
  - CSS vars defined in layout.js via next/font, aliased in globals.css

### Data Management Pattern
All content lives in flat JS/JSX files in `src/data/`:
- `blogPosts.js`: Blog posts with inline content strings, hierarchical file-tree metadata, helper functions for filtering
- `projects.js`: Portfolio work items with metadata + helper functions
- `stashItems.js`: Curated inspiration collection (music, books, art, quotes) with categories
- **NO CMS, NO API routes** — statically generated, markdown content inline as strings in JS

### Blog Architecture (File-Tree UI)
Blog uses Radix UI Accordion-based file-tree navigation (`src/components/reframe-ui/file-tree.jsx`):
1. `blogFileTree` in `blogPosts.js` defines hierarchical folder structure
2. `populateFileTree()` helper dynamically injects posts into tree based on `categories` array
3. `BlogSidebar.jsx` renders tree with custom bitmap folder/checkbox icons
4. Desktop: Accordion-based tree with nested folders
5. Mobile: Horizontal chips for category filtering
6. **Sound design**: Checkbox toggle plays 220Hz square wave (0.09s) via Web Audio API

### Component Patterns
- **Server Components**: Default for page.js files (Home, Work, Blog pages)
- **Client Components**: Explicitly marked with `'use client'` — used for interactivity (Header nav, StashMasonry, BlogSidebar)
- **No prop-types**: JavaScript-only, document params via JSDoc comments where helpful
- **Path aliases**: `@/*` maps to `src/*` (see jsconfig.json)

### Animation & Interaction
- **Framer Motion**: Primary animation library (see FadeIn.jsx, PageFade.jsx)
- **GSAP**: Available but use sparingly — prefer Framer Motion for React patterns
- **Interaction philosophy**: Subtle, purposeful — no flashy effects unless conceptually justified

## Key Conventions

### CSS Architecture
1. **Scoped by feature**: `.museum-*` for homepage/global, `.words-*` for blog, `.stash-*` for stash page
2. **Typography scale**: Defined in globals.css with clamp() for fluid responsive sizing (display-hero, display-xl, heading-*, body-* classes)
3. **Color system**: CSS vars in `:root` — accent colors inspired by Stedelijk (yellow, green, red)
4. **Editorial accents**: 8px thick horizontal rules (`.museum-separator`, `.words-rule`), generous negative space

### File Structure Rules
- Routes: `src/app/[route]/page.js` (App Router structure)
- Components: `src/components/` — flat except for feature folders (home/, about/, ui/, reframe-ui/)
- Shared utilities: `src/lib/utils.js` (clsx + tailwind-merge helper)
- Static assets: `public/` with organized subfolders (fonts/, icons/, home-images/, about-images/)

### Content Patterns
Blog post structure in `blogPosts.js`:
```javascript
{
  id: "slug",
  title: "Title",
  slug: "slug",
  date: "2025.9.17", // YYYY.M.D format
  author: "Brennan K.A. Pollock",
  readingTime: "8 min read",
  categories: ["parent-folder", "child-folder"], // Maps to file tree
  excerpt: "One-line description",
  content: `Markdown-style content string`
}
```

### State Management
- **No global state library**: Use React hooks (useState, useContext) where needed
- **URL state for filters**: Blog/stash pages use client-side filtering, no query params yet
- **Local storage**: Avoided for now — fresh state on each visit

## Development Workflow

### Commands (npm only)
```bash
npm run dev        # Next.js dev server with Turbopack (localhost:3000)
npm run build      # Production build (also uses Turbopack)
npm start          # Serve production build
npm run lint       # Biome check (linter)
npm run format     # Biome format --write
```

### Code Style (Biome)
- **2-space indentation**
- **Recommended rules** for Next.js + React enabled
- **Organize imports** on save (via assist.actions.source)
- **No semicolons** enforced — let Biome decide
- Ignore `noUnknownAtRules` for Tailwind v4 syntax

### Design Tokens Philosophy
From `docs/portfolio-design-language.md`:
- **Asymmetrical balanced layouts** with generous white space
- **Typography as primary design element** — hierarchy through size, weight, spacing
- **Strategic color usage** — neutral base, minimal accent colors
- **Content-first approach** — layout adapts to content, not rigid templates

## Common Pitfalls

1. **Don't create tailwind.config.js** — Tailwind v4 uses inline @theme declarations in CSS
2. **Don't use @apply in CSS** — Use vanilla CSS or Tailwind classes directly in JSX
3. **Don't add TypeScript** — Project deliberately uses JavaScript for simplicity
4. **Client component boundaries**: Mark minimal scope with 'use client', keep most components server-side
5. **Image optimization**: Use Next.js `<Image>` component for photos, but `<img>` acceptable for icons/small assets (especially if unoptimized bitmap style)
6. **File tree metadata**: When adding blog posts, ensure `categories` array matches `blogFileTree` structure

## Design System Patterns

### Bitmap Aesthetic
- Custom bitmap-style folder/checkbox icons in `/public/icons/`
- Pixelated, retro computing feel for file tree UI
- `unoptimized` flag on Next Image for bitmap assets to preserve pixel art

### Editorial Layout Patterns
1. **Museum hero**: Edge-to-edge titles with `.display-hero-xl` typography (clamp 6rem → 20rem)
2. **Rule bands**: 8px horizontal dividers (`.museum-separator`)
3. **Marquee lines**: Repeating exhibition-style text with inline images (see homepage)
4. **Masonry grids**: Stash page uses CSS Grid masonry with infinite scroll (Intersection Observer)

### Reading Experience
Blog articles use Departure Mono for body text (`--font-words`) — technical, readable, distinctive. Generous line-height, constrained measure (~70ch), clear heading hierarchy.

## External Inspirations
Referenced in `docs/`:
- **Kurt Champion**: Visual confidence, large-scale typography
- **FREAK MAG**: Organized cultural flair, contributor-driven
- **Stripe Dev Blog**: Developer-first readability, technical polish
- **Stedelijk Museum**: Editorial accent colors, systematic design
- **Departure Mono**: Monospace as primary typeface, technical aesthetic

---

**Note**: See CLAUDE.md for detailed technical context, docs/style-guide.md for visual specifications, and docs/portfolio-design-language.md for design philosophy deep-dive.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brennankapollock) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
