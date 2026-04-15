# Copilot Instructions for AI Coding Agents

## Project Overview
Probst Excavating is a Local SEO-optimized blog and marketing website built with **Astro + React + Tailwind CSS**. The site represents a family-owned excavation business serving Aurora, IN and the Tri-State area (Indiana, Ohio, Kentucky) for 20+ years.

**Primary Goals:**
- Local SEO optimization for Aurora/Lawrenceburg, Indiana and surrounding areas
- Blog-focused content marketing featuring excavation services, tips, and project showcases
- Migration from existing WordPress site at https://probstexcavating.com/

## Architecture & Tech Stack

**Framework:** Astro 5 with React integration
- **Why Astro:** Optimized for content-heavy sites with excellent SEO and performance
- **React Usage:** Interactive components only (forms, UI widgets) - most content is static Astro
- **Styling:** Tailwind CSS v4 via Vite plugin

**Key Directories:**
- `src/layouts/` - BaseLayout with comprehensive SEO meta tags & Local Business Schema
- `src/pages/` - File-based routing; index and blog/[slug] pages
- `src/content/blog/` - Markdown blog posts with frontmatter schema validation
- `src/components/` - React components for interactive features
- `src/styles/global.css` - Tailwind imports and global styles

## Local SEO Strategy

**Schema.org Implementation:**
- Local Business JSON-LD in [BaseLayout.astro](src/layouts/BaseLayout.astro)
- Structured data includes: business name, address (Aurora, IN), service areas, contact info
- Update schema in BaseLayout when business details are finalized (phone, full address)

**Content Schema:**
- Blog posts include `location` and `service` fields for local targeting
- Tags should reference local areas and specific services
- Example: `tags: ['aurora indiana', 'sewer repair', 'dearborn county']`

**Meta Tags:**
- Every page has unique title, description, canonical URL
- Open Graph and Twitter Card support for social sharing
- Sitemap auto-generated via `@astrojs/sitemap` integration

## Content Workflows

**Adding Blog Posts:**
1. Create `.md` file in `src/content/blog/`
2. Use frontmatter schema from `src/content/config.ts`
3. Include: title, description, pubDate, tags, location (for local SEO)
4. Optional: heroImage, service, updatedDate

**Blog Post Schema Requirements:**
```yaml
---
title: string (required)
description: string (required)
pubDate: date (required)
heroImage: string (optional, /blog/image.jpg)
tags: array (default: [])
location: string (optional, for local SEO)
service: string (optional, e.g., "Sewer Line Repair")
---
```

## Development Commands

- `npm run dev` - Start dev server (http://localhost:4321)
- `npm run build` - Production build (outputs to `dist/`)
- `npm run preview` - Preview production build locally

## SEO Best Practices for This Project

1. **Always include location keywords** in blog titles and descriptions (Aurora, Lawrenceburg, Dearborn County, Tri-State)
2. **Service-specific content** - Each blog post should target a specific service when possible
3. **Internal linking** - Link between blog posts and service sections
4. **Image optimization** - Use WebP format, include alt text with location/service keywords
5. **Schema updates** - Keep Local Business schema in BaseLayout current with accurate business info

## Key Files to Reference

- [astro.config.mjs](astro.config.mjs) - Site URL and integrations
- [src/layouts/BaseLayout.astro](src/layouts/BaseLayout.astro) - SEO meta tags, schema, global structure
- [src/content/config.ts](src/content/config.ts) - Content schema definitions
- [src/pages/index.astro](src/pages/index.astro) - Homepage with services overview
- [src/pages/blog/[slug].astro](src/pages/blog/[slug].astro) - Dynamic blog post template

## Service Areas & Keywords
Target locations: Aurora, Lawrenceburg, Dearborn County, IN; Tri-State area (IN/OH/KY)
Core services: Excavation, Land Clearing, Sewer Line Repair, Water Lines, Storm Drainage, Site Preparation, Hydroseeding

## Next Steps for Development
- Add contact form with location-based routing
- Implement service-specific landing pages
- Create portfolio/gallery section for project showcases
- Add Google Business Profile integration
- Set up analytics and Search Console tracking

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/amwaltamath) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
