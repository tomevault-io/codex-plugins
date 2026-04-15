# hlc23.github.io — Quick AI agent guide

Purpose: give an AI coding agent the minimal, actionable knowledge to be productive in this Hugo-based CTF blog repository.

- Project type: Hugo static site (Hugo extended) with two themes: `enchanted-lowkey` (blog) and `reveal-hugo` (presentations). Themes live under `themes/` as git submodules — do NOT edit them directly.
- Languages: bilingual site (default `zh-tw`). Content files commonly appear as `index.md` (zh-tw) and `index.en.md` (English).

- Quick dev commands (Windows PowerShell):
  - npm install  # install PostCSS/Tailwind tooling
  - hugo server  # local dev with live reload
  - hugo --minify  # production build -> outputs to `public/`

- Important config & integration points (specific files):
  - `config/_default/hugo.toml` — base config, theme array, goldmark `unsafe = true` for Reveal slides
  - `config/_default/menus.toml`, `menus.en.toml` — navigation per-language
  - `config/tailwind.config.js` and `config/postcss.config.js` — Tailwind & PostCSS pipeline used by Hugo Pipes
  - `layouts/partials/head/styles.html` — concatenates and processes CSS via Hugo Pipes
  - `assets/css/custom.css` (override theme styles) and `assets/css/spoiler.css` (project overrides)
  - `layouts/_markup/render-heading.html` — custom heading renderer that injects anchor links
  - `layouts/shortcodes/spoiler.html`, `layouts/shortcodes/alert.html` — common shortcodes used across posts

- Content & archetypes: use `hugo new` to scaffold. Archetypes live under `archetypes/` (e.g. `archetypes/posts.md`, `archetypes/slides.md`). Examples:
  - Blog post: `hugo new content/posts/my-post/index.md` (edit front matter: `draft:false`, `tableOfContents:true/false`)
  - Reveal slide bundle: `hugo new content/slides/my-presentation/_index.md` and set `outputs = ["Reveal"]`

- Reveal.js specifics:
  - Slides require front matter `outputs = ["Reveal"]` and site Goldmark `unsafe = true` (see `config/_default/hugo.toml`).
  - Use `---` for horizontal slides and `___` for vertical sections.

- Build & deploy notes:
  - Repo has GitHub Actions deploy workflow at `.github/workflows/hugo.yml` (builds with Hugo extended and publishes to Pages).
  - Themes are submodules — update via `git submodule update --remote --merge` when necessary.

- Conventions & patterns to follow (discoverable from repo):
  - Images and auxiliary files live next to each post's `index.md` (content bundles). Use relative paths in markdown.
  - Keep UI/style overrides in `assets/css/custom.css` instead of editing `themes/`.
  - Tailwind purge scans content and theme files; add new dynamic classes to `config/tailwind.config.js` if they are stripped in production.

- Quick checks before editing/building:
  1. Run `npm install` once (PostCSS/Tailwind). 2. Confirm Hugo extended is available locally. 3. For slide changes, ensure `unsafe=true` in goldmark.

If anything above is unclear or you want a shorter or longer agent brief, tell me which sections to expand or trim and I will iterate.
# hlc23.github.io - AI Coding Agent Instructions

## Project Overview
This is a **bilingual Hugo static site** (繁體中文/English) for a CTF blog with integrated Reveal.js presentation support. The site uses two Hugo themes: `enchanted-lowkey` (main blog) and `reveal-hugo` (presentations).

## Architecture

### Dual Theme Setup
- **Primary theme**: `enchanted-lowkey` - customized blog theme with Tailwind CSS
- **Secondary theme**: `reveal-hugo` - for creating presentation slides
- Both themes coexist; content type determines which renders
- Theme array in config: `theme = ['enchanted-lowkey', 'reveal-hugo']`

### Directory Structure
```
config/_default/        # Split Hugo configuration
  hugo.toml            # Base config (baseURL, languages, theme)
  menus.toml           # zh-tw navigation menu
  menus.en.toml        # English navigation menu
  params.toml          # Site parameters (avatar, favicon, comments)
content/
  posts/               # Blog posts (CTF writeups, markdown guides)
    */index.md         # Each post is a directory with resources
  slides/              # Reveal.js presentations (unused currently)
  about/               # About pages per language
layouts/
  shortcodes/          # Custom Hugo shortcodes
  _markup/             # Custom markdown renderers
  partials/head/       # CSS/JS injection points
assets/css/            # Source CSS files (processed by Hugo Pipes)
config/                # PostCSS & Tailwind configs
themes/                # Git submodules (DO NOT EDIT DIRECTLY)
public/                # Generated static site (gitignored)
i18n/                  # Translation strings per language
```

## Critical Developer Workflows

### Local Development
```bash
# Install dependencies (required once)
npm install

# Start Hugo dev server with live reload
hugo server

# Build for production (outputs to public/)
hugo --minify
```

### Creating New Content
```bash
# New blog post (uses archetypes/posts.md template)
hugo new content/posts/my-new-post/index.md

# Edit front matter to publish:
# draft: false
# tableOfContents: true/false
# categories: [CTF, Programming, etc.]
# tags: [Write Up, Tutorial, etc.]

# New Reveal.js presentation (single file)
hugo new content/slides/my-presentation.md

# New Reveal.js presentation (directory with multiple files)
hugo new content/slides/my-presentation/_index.md
# This creates a bundle with _index.md and additional-content.md
```

### Deployment
- **GitHub Actions**: Automatic deployment to GitHub Pages on push to `main` branch
- Workflow: `.github/workflows/hugo.yml`
- Hugo version: 0.114.0 (extended)
- Production build runs: `hugo --minify --baseURL "${{ steps.pages.outputs.base_url }}/"`

## Project-Specific Conventions

### Bilingual Content Pattern
- Every content file has two versions: `index.md` (zh-tw) and `index.en.md` (en)
- Default language is `zh-tw` (traditional Chinese)
- Language-specific menus in `config/_default/menus.toml` and `menus.en.toml`
- Translation strings in `i18n/zh-tw.toml` and `i18n/en.toml`

### Custom Shortcodes
```markdown
# Spoiler text (click to reveal)
{{<spoiler>}}Hidden content here{{</spoiler>}}

# Alert boxes (info/warning/error/success)
{{<alert "info" "Optional Title">}}Alert content{{</alert>}}
```

Implementation files:
- `layouts/shortcodes/spoiler.html` - JavaScript-based click reveal
- `layouts/shortcodes/alert.html` - SVG icons + severity classes
- `assets/css/spoiler.css` - Spoiler styling with light/dark theme support

### Custom Heading Renderer
All headings automatically get anchor links with `#` symbol prepended:
```html
<!-- Rendered as: -->
<h1 id="section-name"><a href="#section-name">#</a> Section Name</h1>
```
Defined in: `layouts/_markup/render-heading.html`
Styled in: `assets/css/custom.css` (emerald green links, underline on hover)

### CSS Architecture
1. **Theme base**: `themes/enchanted-lowkey/assets/css/tailwind.css` (auto-loaded)
2. **Spoiler styles**: `assets/css/spoiler.css`
3. **Custom overrides**: `assets/css/custom.css` (your modifications)
4. **Processing pipeline**: `layouts/partials/head/styles.html` concatenates all CSS → PostCSS → Tailwind purge → minify (production)

**Key**: Never edit theme files directly. Override in project root `assets/css/custom.css` using `@layer components` or `@layer utilities`.

### Tailwind Configuration
- Config: `config/tailwind.config.js`
- Scans: themes, layouts, content (markdown), public HTML
- Dark mode: `class` based (toggle via `.dark` class)
- Extended colors: `custom-blue`, `custom-green`
- Custom font: `font-sans` with Inter + system fonts fallback

### CTF Writeup Patterns
Observed in `content/posts/K17CTF_Write_Up/index.md`:
- **Images co-located**: Store challenge images in same directory as `index.md`
- **Markdown references**: `![alt text](image.png)` (relative paths)
- **Solution scripts**: Include Python/shell scripts in post directory
- **Code blocks**: Use triple backticks with language hints
- **Flag format**: Always use code formatting: `` `K17{...}` ``
- **Challenge metadata**: Tag with category `CTF`, tags like `Write Up`

### Front Matter Templates

#### Blog Posts (`archetypes/posts.md`)
```yaml
---
title: "Post Title"
date: 2025-01-01T00:00:00+08:00
draft: true              # Set false to publish
tableOfContents: false   # Enable TOC sidebar
description: ''
categories:
  - CTF                  # Main category
tags:
  - Write Up             # Multiple tags allowed
---
```

#### Reveal.js Presentations (`archetypes/slides.md` or `archetypes/slides/_index.md`)
```markdown
+++
title = "Presentation Title"
date = 2025-10-15T00:00:00+08:00
outputs = ["Reveal"]
[reveal_hugo]
theme = "black"          # black, white, league, beige, sky, night, serif, simple, solarized
highlight_theme = "monokai"
transition = "slide"     # none, fade, slide, convex, concave, zoom
transition_speed = "default"
+++

# First Slide

Content here

---

# Second Slide

Use `---` to separate horizontal slides
Use `___` to separate vertical slides
```

## Integration Points

### Hugo Pipes (Asset Processing)
- PostCSS config: `config/postcss.config.js`
- Pipeline: Tailwind CSS → Autoprefixer → cssnano (production only)
- Trigger: `postCSS` function in `layouts/partials/head/styles.html`
- Environment detection: `hugo.Environment` (development vs production)

### MathJax Integration
Configured in `layouts/partials/head/styles.html`:
- CDN: MathJax 2.7.1
- Inline delimiters: `$...$` and `\(...\)`
- For equations in markdown posts

### External Dependencies
- **Themes**: Git submodules in `themes/` directory
  - Update with: `git submodule update --remote --merge`
- **Node packages**: `package.json` (Tailwind, PostCSS, autoprefixer, cssnano)
- **Hugo extended**: Required for SCSS/PostCSS processing

## Reveal.js Presentation Workflow

### Configuration Requirements
In `config/_default/hugo.toml`, you need:
```toml
theme = ['enchanted-lowkey', 'reveal-hugo']  # Multiple themes

[markup.goldmark.renderer]
  unsafe = true  # Required for reveal-hugo

[outputFormats.Reveal]
  baseName = "index"
  mediaType = "text/html"
  isHTML = true
```

### Creating Presentations
1. Create a directory: `content/slides/my-presentation/`
2. Add `_index.md` with front matter: `outputs = ["Reveal"]`
3. Separate slides with `---` (horizontal) or `___` (vertical)
4. Access at: `http://localhost:1313/slides/my-presentation/`

### Slide Organization
- **Single file**: All slides in `_index.md`
- **Multiple files**: Split across `content/slides/my-presentation/*.md`
  - Use `weight` parameter to control order
  - Files without `weight` are ordered alphabetically

### Available Reveal.js Themes
`black`, `white`, `league`, `beige`, `sky`, `night`, `serif`, `simple`, `solarized`

### Presentation Controls
- Arrow keys: Navigate slides
- `Esc` or `O`: Overview mode
- `S`: Speaker notes
- `F`: Fullscreen
- `B` or `.`: Pause (blackout)

## Common Pitfalls

1. **Theme editing**: Changes in `themes/` will be overwritten. Always override in project root.
2. **Draft posts**: Set `draft: false` in front matter to publish. Draft posts don't appear in production builds.
3. **Language files**: When adding content, create both `.md` and `.en.md` versions for full bilingual support.
4. **Asset paths**: In markdown, use relative paths for images. Hugo resolves them correctly.
5. **Tailwind purging**: Unknown classes will be purged in production. Add to `content` array in `tailwind.config.js` if needed.
6. **PostCSS errors**: Ensure `npm install` is run before building. Hugo Pipes needs these dependencies.
7. **Reveal.js output**: Must add `outputs = ["Reveal"]` in front matter, or presentation won't render.
8. **Unsafe HTML**: Reveal.js requires `unsafe = true` in goldmark renderer config.

## Key Files Reference

| Purpose | File |
|---------|------|
| Site config | `config/_default/hugo.toml` |
| Navigation menus | `config/_default/menus.toml`, `menus.en.toml` |
| Site metadata | `config/_default/params.toml` |
| Custom CSS | `assets/css/custom.css` |
| Tailwind config | `config/tailwind.config.js` |
| Post template | `archetypes/posts.md` |
| Slides template (single) | `archetypes/slides.md` |
| Slides template (bundle) | `archetypes/slides/_index.md` |
| Spoiler shortcode | `layouts/shortcodes/spoiler.html` |
| Alert shortcode | `layouts/shortcodes/alert.html` |
| Heading renderer | `layouts/_markup/render-heading.html` |
| CSS pipeline | `layouts/partials/head/styles.html` |

## Style Guide

### Markdown
- Use `##` for section headers (not `#`, which is post title)
- Code blocks: Use fenced code blocks with language hints
- Links: Prefer descriptive text over raw URLs
- Images: Alt text required for accessibility

### Naming Conventions
- Posts: lowercase with hyphens (`my-post-title`)
- Categories: Title Case (`CTF`, `Programming`)
- Tags: Title Case (`Write Up`, `Tutorial`)

### Chinese/English Mixing
- Use English for technical terms (Hugo, Tailwind, CTF)
- Use Chinese for narrative content
- Code/command examples always in English

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hlc23) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
