# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a personal Hugo static site (twobiers.github.io) with a fully custom layout and the [hugo-admonitions](https://github.com/KKKZOZ/hugo-admonitions) Hugo module. Content is written in Markdown or AsciiDoc. The site is deployed to GitHub Pages on every push to `main`.

## Development Commands

**Local dev server (Docker):**
```bash
docker compose up
# Site available at http://localhost:1313
```

**Local dev server (native Hugo required):**
```bash
hugo server
```

**Build for production:**
```bash
hugo --minify
```

Hugo version: `0.152.2` (extended). AsciiDoc support requires `asciidoctor` and `asciidoctor-chroma` gems (`gem install asciidoctor asciidoctor-chroma`).

## Content Structure

- `content/posts/` — long-form blog posts, each in its own directory (e.g. `2024-08-28_conditional_git_author/index.md`)
- `content/dev/` — technical/dev-focused articles
- `content/random/` — short-form notes (no longer synced from Supernotes; managed locally)
- `content/about/` — about page

**Date inference:** Hugo derives the date from the filename prefix (`YYYY-MM-DD`) when not set in frontmatter (`frontmatter.date = [":default", ":filename", ":fileModTime"]`).

## Random Posts

To create a new random post locally, run:
```bash
./random.sh
```

This opens today's file (`content/random/YYYY-MM-DD.md`) in your `$VISUAL`/`$EDITOR`.

## Configuration

- `config/_default/` — base config split across `config.toml`, `params.toml`, `menus.en.toml`, `markup.toml`, `module.toml`, `taxonomies.toml`
- `config/production/` — production overrides
- Hugo modules are used (not git submodules) for the theme; `go.mod`/`go.sum` manage them

## Theme & Customization

- Layout: fully custom (Congo removed; only active Hugo module is `hugo-admonitions`)
- Color scheme: custom avocado green palette, defined via CSS variables in `assets/css/main.css`
- CSS: `assets/css/main.css` (primary), `assets/css/custom.css` (overrides), `assets/css/syntax-dark.css` / `assets/css/syntax-light.css` (code highlighting)
- JS: `assets/js/theme.js` (dark/light toggle), `assets/js/code-copy.js` (copy button)
- Custom layouts in `layouts/`:
  - `baseof.html`, `index.html` — base shell and home page
  - `_partials/` — `header.html`, `footer.html`, `head.html`, `profile.html`, `pagination.html`, `article-meta.html`, `author-links.html`, `icon.html`
  - `_partials/home/custom.html` — home page content
  - `about/list.html` — about page
  - `random/list.html` — random notes list
  - `shortcodes/` — `details`, `email-link`, `experience`, `skill-group`, `spotify`
- AsciiDoc syntax highlighting uses Chroma via the `asciidoctor-chroma` extension; highlight classes are generated (not inline styles — `noClasses = false`)
- Goldmark unsafe mode is enabled (raw HTML in Markdown is allowed)

## CI/CD

- `.github/workflows/gh-pages.yml` — builds and deploys to GitHub Pages on push to `main`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/twobiers) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
