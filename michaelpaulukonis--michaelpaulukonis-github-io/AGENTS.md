# GitHub Copilot Instructions

## Project Context

This is my personal website and blog, built as a static site using **Nuxt 4** and **Nuxt Content v3**.
The project is in a transitional state but heavily migrated. The goal is to enforce Nuxt 4 and Content v3 patterns strictly and avoid legacy Nuxt 3 / Content v2 code.

## Technology Stack

- **Framework:** Nuxt 4 (via `compatibilityVersion: 4`)
- **Content Management:** Nuxt Content v3
  - **Database:** SQLite (via `better-sqlite3`) for development; static JSON for production.
  - **Configuration:** `content.config.ts` is the source of truth for collections.
- **Styling:** Tailwind CSS (v6.14.0), CSS/SCSS
- **Template Engine:** Pug (via `@vue/language-plugin-pug`)
- **Package Manager:** pnpm
- **Other:** `@vueuse/core`, `@nuxt/fonts`

## Project Structure

The project follows the **Nuxt 4 directory structure**, utilizing the `app/` directory for source code.

- `content.config.ts` — **CRITICAL**: Definitions for content collections (`content`, `blog`, `authors`) and Zod schemas.
- `app/` — Main application source.
  - `app/pages/` — Route components. MUST use `queryCollection`.
  - `app/components/` — Reusable Vue components.
  - `app/layouts/` — Page layouts.
  - `app/app.vue` — Root component.
- `content/` — Markdown/JSON content files (Data Source).
- `public/` — Static assets (images, favicons). All content assets must live here.
- `docs/` — Documentation and architectural snapshots.

## Coding Standards & Patterns

### Data Fetching (Content v3)

- **STRICT PROHIBITION**: **NEVER** use `queryContent()` or `<ContentDoc />`. These are legacy Content v2 patterns.
- **Primary API**: Use `queryCollection('<collection-name>')`.
- **Filtering**: Use SQL-like syntax (Content v3 standard).
  - **Correct**: `.where('tags', 'LIKE', '%pinned%')`
  - **Incorrect** (Legacy): `.where({ tags: { $contains: 'pinned' } })`
- **Single Items**: Use `.first()` instead of `.findOne()`.
- **Surround**: Use `queryCollectionItemSurroundings` for next/prev links.

### Rendering

- Use `<ContentRenderer :value="data" />` to render fetched content.
- Ensure data is fetched using `useAsyncData` with a unique key.

### Templating & Styling

- **Pug**: Use Pug syntax for Vue templates (`<template lang="pug">`).
- **Tailwind**: Use utility classes for styling.

### Asset Management

- **Location**: Store images and static assets in `public/`.
- **Referencing**: In Markdown, use absolute paths relative to the root (e.g., `![Alt](/images/blog/post.jpg)`).
- **Legacy**: Do NOT expect assets to be co-located with Markdown files in `content/`.

## Development Workflow

1. **Schema Updates**: Modify `content.config.ts` when changing content structure.
2. **Local Dev**: Run `pnpm dev`.
3. **Production Build**: Run `pnpm generate` for Static Site Generation (SSG).
4. **Deployment**: Deploys to GitHub Pages via `gh-pages` (using the `deploy` script).

## Reference Materials

- [Nuxt Content v3 Documentation](https://content.nuxt.com)
- Repository Snapshot: `docs/reference/2025-12-21-repository-snapshot.md`

## Notes for Copilot

- **Nuxt 4 Mode**: The project uses `future: { compatibilityVersion: 4 }`. Always look for application code in `app/`, not the root.
- **Migration Status**: You may see legacy files or patterns in the history. Ignore them. Always prefer the patterns defined in this file.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MichaelPaulukonis)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/MichaelPaulukonis)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
