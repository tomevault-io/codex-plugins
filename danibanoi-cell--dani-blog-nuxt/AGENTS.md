# Dani Banoi Photography Portfolio - AI Agent Instructions

## Project Overview
Nuxt 4 portfolio website for Verona-based photographer specializing in travel/documentary photography. Built with MySQL backend, JWT auth, i18n (IT/EN), and dark/light theme support.

## Architecture & Key Concepts

### Photo Session Model
- **Session-based organization**: Photos are grouped by `session_slug` column in `photos` table
- `/session/[slug]` route displays all photos with matching `session_slug`
- Dynamic schema migration: API endpoints check for `session_slug` column existence and add it if missing (see `server/api/sessions/[slug].get.ts:4-8`)
- Fallback compatibility: If no session found, falls back to single photo by slug

### Database Layer (`server/utils/db.ts`)
- MySQL connection pooling via `mysql2/promise`
- Centralized `query()` helper: All DB operations use `await query(sql, params)` 
- Config via env vars: `DB_HOST`, `DB_USER`, `DB_PASSWORD`, `DB_NAME` (defaults to `dani_portfolio`)
- Schema: See `database/schema.sql` - photos, users, tags, photo_tags pivot table

### Authentication (`server/utils/auth.ts`)
- JWT-based: `generateToken()` returns 7-day tokens
- Protected routes: Use `await requireAuth(event)` in API handlers
- Checks `Authorization: Bearer <token>` header
- Admin routes: `/admin/dashboard`, `/admin/login`

### Theme System (`app/composables/useTheme.ts`)
- Composable pattern: `const { isDark, toggleTheme, setTheme } = useTheme()`
- Persists to localStorage with key `dani-portfolio-theme`
- CSS custom properties in `app/layouts/default.vue:43-67` (`--bg-primary`, `--text-primary`, etc.)
- Dark/light classes on `document.documentElement`

### Internationalization (i18n)
- `@nuxtjs/i18n` module configured in `nuxt.config.ts:14-21`
- Default locale: Italian (`it`) with `prefix_except_default` strategy
- Translation files: `locales/it.json` and `locales/en.json`
- Access via `$t('key.path')` in templates or `t('key.path')` in script

## Development Workflow

### Commands
```bash
npm run dev          # Dev server on http://localhost:3000 (--host for network access)
npm run build        # Production build
npm run preview      # Preview production build
```

### Database Setup
1. Create MySQL database: `dani_portfolio` (or set `DB_NAME` env var)
2. Run schema: `mysql -u root -p dani_portfolio < database/schema.sql`
3. Set env vars in `.env` (not tracked): `DB_HOST`, `DB_USER`, `DB_PASSWORD`, `JWT_SECRET`

### Photo Upload Scripts (`scripts/`)
- `upload-photos.js`: Bulk upload from `public/foto-sito/` directory
- `upload-alice-sessions.js`: Upload photos grouped by session
- `upload-all-alice-albums.js`: Upload complete album collections
- Run with: `node scripts/upload-photos.js` (requires MySQL connection)

### File Upload API (`server/api/photos/index.post.ts`)
- Uses `formidable` for multipart form data
- Saves to `public/uploads/` directory
- Auto-generates slug from title
- Supports tags (comma-separated), category, session_slug
- **Important**: Checks and adds `session_slug` column dynamically if missing (lines 29-33)

## Code Conventions

### API Route Patterns
- GET lists: `server/api/{resource}/index.get.ts` - Returns array with success wrapper
- GET single: `server/api/{resource}/[id].get.ts` or `[slug].get.ts`
- POST create: `server/api/{resource}/index.post.ts`
- PUT update: `server/api/{resource}/[id].put.ts`
- DELETE: `server/api/{resource}/[id].delete.ts`
- Auth required: Call `await requireAuth(event)` at handler start

### Component Patterns
- **Layouts**: Single `default.vue` with `<NuxtPage />` slot, includes HeaderBar, SidebarNav, FooterBar, CookieConsent
- **Page structure**: Always wrap in `<div class="page-container">` → `<main class="main-content" role="main">` for consistent styling
- **Session pages**: Use `route.params.slug` to fetch photos, shuffle array for display randomization
- **Lightbox**: Fullscreen overlay with prev/next navigation (see `app/pages/session/[slug].vue`)

### Styling Approach
- **No Tailwind utilities**: Despite `@nuxtjs/tailwindcss` installed, project uses custom CSS in `<style scoped>` blocks
- **Theme variables**: Reference CSS custom props (`var(--bg-primary)`, `var(--text-primary)`, etc.)
- **Transitions**: Use Vue `<Transition name="lightbox">` with matching CSS classes
- **Fonts**: Google Fonts (Oswald, Roboto) loaded in `nuxt.config.ts:68-73`, custom fonts in `app/assets/styles/fonts.css`

### SEO Implementation
- Comprehensive meta tags in `nuxt.config.ts:27-64`
- Italian-first (`lang="it"`) with alternate locales
- Geographic tags for Verona/Valpolicella/Lago di Garda region
- Schema.org microdata: `itemscope`, `itemtype`, `itemprop` attributes in templates
- Semantic HTML: `<main role="main">`, `<section aria-labelledby>`

## Critical Integration Points

### Photo Query Patterns
```typescript
// Get all published photos with tags
const photos = await query(`
  SELECT p.*, GROUP_CONCAT(t.name) as tags
  FROM photos p
  LEFT JOIN photo_tags pt ON p.id = pt.photo_id
  LEFT JOIN tags t ON pt.tag_id = t.id
  WHERE p.published = 1
  GROUP BY p.id
  ORDER BY p.created_at DESC
`)
```

### Session Fetching Logic
1. Query by `session_slug` first
2. If no results, fallback to single photo by slug
3. Always return `{ success: true, photos: [], session: { slug } }` structure

### Dynamic Schema Migrations
Always check for column existence before querying:
```typescript
const columns = await query("SHOW COLUMNS FROM photos LIKE 'session_slug'")
if (!Array.isArray(columns) || columns.length === 0) {
  await query("ALTER TABLE photos ADD COLUMN session_slug VARCHAR(255) NULL, ADD INDEX idx_session (session_slug)")
}
```

## Project-Specific Gotchas

1. **Session slug is optional**: Some photos have `session_slug`, others only have unique `slug`
2. **Auto-migration on API calls**: Schema changes happen at runtime, not via migrations
3. **Image paths**: Use `/foto-sito/` for public gallery images, `/uploads/` for admin uploads
4. **No TypeScript types for DB**: Queries return `any[]`, manual type assertions needed
5. **i18n strategy**: Default locale (IT) has no prefix, EN routes prefixed with `/en/`
6. **Cookie consent**: GDPR-compliant banner in `CookieConsent.vue` component
7. **Dev server**: Runs with `--host` flag for network access during development

---
> Source: [danibanoi-cell/dani-blog-nuxt](https://github.com/danibanoi-cell/dani-blog-nuxt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-06 -->
