# Copilot Instructions for KeyKey Kitchen

## Project Overview

KeyKey Kitchen is a SvelteKit 5 application for a Jakarta-based dark kitchen business, deployed on Cloudflare Workers. The site features a multilingual (English/Indonesian) weekly menu system with ordering via WhatsApp/Instagram.

## Architecture & Tech Stack

- **Frontend**: SvelteKit 5 with Tailwind CSS 4, deployed to Cloudflare Workers
- **Database**: SQLite with Drizzle ORM (`src/lib/server/db/`)
- **Internationalization**: Paraglide.js with Inlang project for EN/ID locales
- **Build**: Bun package manager, Vite bundler, Wrangler for Cloudflare deployment

## Key Developer Workflows

```bash
# Development
bun run dev                 # Start dev server
bun run preview            # Build + preview with Wrangler
bun run deploy             # Deploy to Cloudflare

# Database operations
bun run db:push           # Push schema changes
bun run db:generate       # Generate migrations
bun run db:studio         # Open Drizzle Studio

# Code quality
bun run check             # Type checking
bun run lint              # ESLint + Prettier
```

## Project-Specific Patterns

### Internationalization with Paraglide
- Messages in `messages/{locale}.json` are auto-generated to `src/lib/paraglide/`
- `hooks.server.ts` handles locale middleware and injects `%paraglide.lang%` into HTML
- Use generated message functions from `$lib/paraglide/messages`

### Cloudflare Workers Integration
- `wrangler.jsonc` configures the Worker deployment
- `src/worker-configuration.d.ts` for Cloudflare-specific types
- Use `bun run cf-typegen` to generate Worker types

### Database Schema
- Simple SQLite schema in `src/lib/server/db/schema.ts` (currently minimal user table)
- Environment variable `DATABASE_URL` required for Drizzle

### Component Structure
- Main landing page in `src/routes/+page.svelte` with weekly menu data
- Static menu items defined inline (consider moving to database/API for dynamic updates)
- Tailwind classes for styling with responsive design patterns

## Integration Points

- **Cloudflare Workers**: `@sveltejs/adapter-cloudflare` for deployment
- **Drizzle ORM**: Database abstraction with SQLite dialect
- **Paraglide.js**: Type-safe i18n with compile-time message generation
- **External Services**: WhatsApp Business API and Instagram for order management

## Examples

- **i18n Setup**: See `project.inlang/settings.json` and `hooks.server.ts`
- **Database Schema**: `src/lib/server/db/schema.ts`
- **Menu Display**: `src/routes/+page.svelte` (lines 8-26 for data structure)
- **Deployment Config**: `wrangler.jsonc` and `svelte.config.js`

## Development Notes

- Use Bun as the primary package manager and runtime
- Tailwind CSS 4 is used (note the newer `@tailwindcss/vite` plugin)
- SvelteKit 5 syntax (runes-based reactivity if using them)
- Environment variables should be added to both `.env` and Wrangler configuration

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hugomarisco) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
