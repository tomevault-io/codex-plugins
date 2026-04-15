# Copilot Instructions for Red-Arrow.io

## Project Overview

This is a personal portfolio website built with Next.js 16 using the App Router, configured for static export and GitHub Pages deployment. The site features MDX-based articles, dark/light theme support, and a clean, professional design built with Tailwind CSS.

## Architecture & Key Patterns

### Static Export Configuration

- Site is configured for static export (`output: 'export'`) with GitHub Pages deployment
- Production builds target custom domain root (`https://www.red-arrow.io`) without a repository subpath basePath
- Images are unoptimized (`images: { unoptimized: true }`) due to static export limitations

### MDX Article System

- Articles live in `src/app/articles/[slug]/content.mdx` with a `page.tsx` server route wrapper
- Each article exports an `article` object from `content.mdx` with metadata (title, description, author, date)
- Articles use `ArticleLayout` component for consistent styling and navigation
- `src/lib/articles.ts` dynamically imports and sorts articles by date using `fast-glob`

### Component Architecture

- Layout uses a fixed background design pattern with centered content (`max-w-7xl`)
- All components follow the pattern of exporting named functions (not default exports except for pages)
- Client components are explicitly marked with `'use client'` directive
- Shared UI components in `src/components/` follow atomic design principles

### Styling Conventions

- Tailwind CSS with custom color system using CSS variables for theming
- Dark mode support via `next-themes` with `class` strategy
- Typography handled by `@tailwindcss/typography` plugin for article content
- Consistent spacing using Tailwind's spacing scale (mt-16, lg:mt-32 patterns)

## Development Workflows

### Local Development

```bash
npm run dev          # Start development server
npm run build        # Build for production
npm run typecheck    # Validate TypeScript
npm run lint         # Run lint checks
```

### Adding New Articles

1. Create directory: `src/app/articles/[slug]/`
2. Add `content.mdx` with required `article` export object and article body
3. Add `page.tsx` that imports `content.mdx`, exports `metadata`, and renders `ArticleLayout`
4. Include images in the same directory
5. Articles auto-appear on `/articles` page via dynamic import system

### RSS Feed Generation

- RSS feed generated at `/feed.xml` via route handler in `src/app/feed.xml/route.ts`
- Automatically includes all articles with proper metadata extraction
- Uses `cheerio` for HTML content processing and `feed` library for RSS generation

## Key Integration Points

### Theme System

- `src/app/providers.tsx` sets up `ThemeProvider` for dark/light mode
- Theme state managed globally, affects all components
- Custom color/theme variables are defined in `src/styles/tailwind.css`

### MDX Processing

- `next.config.mjs` configures MDX with `remark-gfm` and `rehype-prism` for syntax highlighting
- `mdx-components.tsx` provides custom component mappings for MDX content
- Prism CSS styling in `src/styles/prism.css` for code blocks

### Canonical URL Management

- `src/lib/site.ts` is the single source of truth for site URL generation
- Feed, sitemap, robots, and metadata rely on this shared helper

### Asset Management

- Images stored in `src/images/` with organized subdirectories (`logos/`, `photos/`)
- Static assets (favicons, etc.) in `src/app/` directory following App Router conventions
- Sharp optimizations disabled for static export compatibility

## Project-Specific Conventions

### File Naming

- Route files: `page.tsx` (use `content.mdx` for article content)
- Components: PascalCase files with named exports (`ArticleLayout.tsx`)
- Utilities: camelCase files in `src/lib/` (`formatDate.ts`, `articles.ts`)

### Import Patterns

- Use `@/` alias for `src/` directory (configured in `tsconfig.json`)
- Component imports use explicit named imports, not default exports
- Type imports use `type` keyword for better tree-shaking

### Responsive Design

- Mobile-first approach with Tailwind breakpoints (sm, lg, xl)
- Consistent container patterns using `Container` component
- Fixed layout with responsive sidebar navigation patterns

## Critical Dependencies

- `@next/mdx` and MDX ecosystem for article content
- `next-themes` for theme switching functionality
- `fast-glob` for dynamic article discovery
- `feed` library for RSS generation
- `@tailwindcss/typography` for article content styling

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tpkreepur) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
