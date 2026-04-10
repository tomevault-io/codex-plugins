
# Mona Mayhem Workspace Instructions

## Project Overview

**Mona Mayhem** is a retro arcade-themed website built with **Astro** that compares GitHub contribution graphs of two users. The project uses Astro as a full-stack framework with server-side rendering via the Node adapter.

- **Framework**: Astro 5.14.4 with `@astrojs/node` adapter
- **Output Mode**: Server-side rendering (SSR)
- **Runtime**: Node.js
- **Key Feature**: GitHub contribution data API endpoints and comparison visualization

## Build & Development Commands

| Command | Purpose |
|---------|---------|
| `npm run dev` | Start local development server (default: http://localhost:3000) |
| `npm run build` | Build for production |
| `npm run preview` | Preview production build locally |
| `npm run astro` | Direct access to Astro CLI |

## Project Structure

```
src/
├── pages/
│   ├── index.astro         # Main homepage
│   └── api/
│       └── contributions/
│           └── [username].ts    # Dynamic API endpoint for GitHub data
docs/                      # Static documentation & step-by-step guide
public/                    # Static assets
```

## Astro Best Practices

### File-Based Routing
- **Pages**: `src/pages/*.astro` → Routes (`.astro` files become dynamic routes)
- **Dynamic Routes**: Use `[param].astro` syntax for dynamic segments (e.g., `[username].ts` for `/api/contributions/octocat`)

### Components & Islands
- Place reusable components in `src/components/` 
- Use `client:*` directives for interactivity: `client:load`, `client:idle`, `client:visible`
- Default: components are static HTML (no JavaScript sent to browser)

### API Endpoints
- `src/pages/api/*.ts` files become server endpoints
- Use dynamic routes: `src/pages/api/[param].ts` → `/api/:param`
- Full Node.js runtime available; use `Astro.request`, `Astro.params`

### Layouts & Templating
- Create layout components in `src/layouts/`
- Use `<slot />` for content injection
- Astro syntax: Mix HTML, JavaScript, and components in `.astro` files

### Static Generation vs. SSR
- **Current Mode**: SSR (`output: 'server'`) via `@astrojs/node`
- Pages are rendered on-demand on the server
- Perfect for dynamic data like GitHub contributions

### Optimization Tips
- **CSS**: Scoped to component by default; use `<style></style>` blocks
- **JavaScript**: Only ship JS for interactive components (`client:*` directives)
- **Images**: Use native `<Image />` component for optimization
- **Hydration**: Use partial hydration to minimize JS payload

## Common Development Workflow

1. Start dev server: `npm run dev`
2. Edit `.astro` files in `src/pages/` for pages/routes
3. Create `.ts` files for API endpoints in `src/pages/api/`
4. Use `Astro.request` and `Astro.params` for dynamic data
5. Restart server if you add new routes or change config

## Key Files

- `astro.config.mjs` — Astro configuration with Node adapter settings
- `tsconfig.json` — TypeScript configuration
- `package.json` — Dependencies and scripts

---

**For detailed Astro docs**: https://docs.astro.build

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alexelabassi)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/alexelabassi)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
