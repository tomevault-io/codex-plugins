# Project Guidelines

## Overview

Vereins-Website for **Reihenhaussiedlung Erding-Ost e.V.** — a German residential association (134 houses in Erding). Single-page Next.js frontend deployed on Vercel. All content is in German (`lang="de"`).

## Tech Stack

- **Framework**: Next.js 15 (App Router) + React 19 + TypeScript (strict)
- **Styling**: Plain CSS with custom properties (design tokens) in `globals.css` — no Tailwind, no component library
- **Font**: Inter via `next/font/google` (self-hosted at build time — never use external font CDNs)
- **Deployment**: Vercel (auto-deploy on push to `main`)
- **CI**: GitHub Actions → typecheck → lint → build (must all pass)

## Build & Test Commands

```bash
npm install          # install dependencies
npm run dev          # local dev server at localhost:3000
npm run typecheck    # tsc --noEmit
npm run lint         # next lint
npm run build        # production build
```

## Project Structure

```
src/app/
  layout.tsx        # Root layout (metadata, font, CSS import)
  page.tsx          # Single-page homepage (all content + data arrays)
  globals.css       # Complete design system (CSS variables, BEM-like classes)
public/
  logo.jpg          # Vereins logo
  lageplan.gif      # Settlement map
docs/
  deployment.md     # Vercel + Strato DNS setup guide
```

## Architecture Decisions

- **Single-page**: All content lives in `page.tsx` as inline data arrays rendered with `.map()`. No database, no API routes, no CMS.
- **CSS-only styling**: BEM-like naming (`.header__inner`, `.news-card`). Design tokens as CSS custom properties (`--c-blue`, `--shadow-md`, `--r-md`).
- **Static generation**: Pages are prerendered at build time. No client-side fetching.
- **No external resources without consent**: Fonts are self-hosted via `next/font`. No CDN links, no external embeds without 2-click pattern.

## Conventions

- Content language is **German** — all UI text, comments in data arrays, and metadata
- Use `next/image` for all images, `next/link` for internal links
- Path alias: `@/*` maps to `./src/*`
- CSS class names follow the pattern: `.block__element--modifier`
- Data arrays (news, board members, documents, etc.) are defined at the top of `page.tsx`

## Legal Requirements (Germany)

This is a German site subject to DDG/DSGVO/TDDDG. See `.github/instructions/de-rechtssicher-abmahnung.instructions.md` for full rules. Key points:
- Impressum must be reachable within 2 clicks
- No external resource loading without consent (fonts, embeds, analytics)
- No tracking or cookies without opt-in consent

## Dev Environment Pitfalls

- **Node.js PATH**: Node may not be in the system PATH. Prepend manually: `$env:Path = "C:\Program Files\nodejs;" + $env:Path`
- **PowerShell execution policy**: Use `npm.cmd` directly instead of `npm` to avoid `.ps1` script blocks
- **Port conflicts**: If port 3000 is busy, Next.js auto-selects the next available port

## Related Documentation

- [docs/deployment.md](docs/deployment.md) — Vercel project setup + Strato DNS migration
- [README.md](README.md) — Quick start and build commands
- `.github/instructions/` — File-scoped instructions for visual quality, SEO, performance, content quality, and German legal compliance

---
> Source: [michi11235813-lgtm/erding-ost](https://github.com/michi11235813-lgtm/erding-ost) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-12 -->
