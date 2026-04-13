# Project Guidelines

## Overview

Personal art portfolio built with [Zola](https://getzola.org) static site generator using the vendored **Zallery** theme (`themes/zallery/`). Images stored in Cloudflare R2, content managed via Sveltia CMS, deployed on Cloudflare Pages.

## Architecture

| Service | Domain | Purpose |
|---------|--------|---------|
| Cloudflare Pages | `artishchow.com` | Static site hosting, auto-deploys from `main` |
| Cloudflare R2 | `images.artishchow.com` | Image storage (public, CORS `*`) |
| Sveltia CMS | `artishchow.com/admin/` | Git-based CMS, GitHub PAT auth, editorial workflow (PRs) |

### Key directories

- `content/artwork/<slug>.md` — artwork posts (TOML frontmatter, flat files)
- `themes/zallery/` — vendored theme with custom modifications for remote images; if a relevant file already exists here, prefer updating it over creating a root-level override unless the change is an intentional site-level replacement
- `static/admin/` — Sveltia CMS (index.html, config.yml)

## Build and Test

```bash
zola serve          # Local dev with hot reload (http://127.0.0.1:1111)
zola build          # Production build (outputs to ./public/)
zola check          # Link checking
```

## Image Handling

All images are hosted on Cloudflare R2 (`https://images.artishchow.com/`). Theme templates (`thumbnail.html`, `img.html` shortcode) detect `http` URLs and render them directly with `crossorigin="anonymous"`.

CMS `preSave` hook and editor component auto-prepend `https://images.artishchow.com/` to bare filenames.

## Conventions

- Artwork posts are flat files in `content/artwork/` for prev/next navigation (`page.lower`/`page.higher`)
- `content/artwork/_index.md` has `transparent = true` — artwork appears at root URL
- Artwork posts use top-level integer `weight` frontmatter and `[extra].day_number`; display metadata should derive from frontmatter, not from slug parsing
- Preserve existing artwork slugs/URLs; new CMS-created artwork slugs use `YYYYMMDD-dN-W-title` where `W` is the 1-indexed `weight`
- Thumbnail images are 800×800 WebP, uploaded to R2
- Image filenames: `YYYYMMDD_DN_Description.webp` (e.g., `20251119_D4_Kasasagi.webp`)
- Config in `config.toml` under `[extra]`: `r2_public_url`, gallery settings, image processing settings

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ishchow)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/ishchow)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
