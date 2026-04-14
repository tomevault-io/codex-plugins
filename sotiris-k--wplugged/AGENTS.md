# WPlugged.com Documentation Site - AI Coding Agent Instructions

## Project Overview
This is an **Eleventy (11ty) static site** documenting WordPress plugins for wplugged.com. It uses Nunjucks templating, Sass/SpruceCss for styling, and a custom navigation system based on frontmatter metadata.

**Not a WordPress project** — the `oldsite/` folder contains legacy WordPress files that are gitignored and should be ignored during development.

## Architecture

### Content Structure
- **Documentation pages**: `src/docs/{plugin-name}/posts/*.md` — individual doc pages
- **Plugin landing pages**: `src/docs/{plugin-name}.md` — front pages with overview/FAQs
- **Homepage**: `src/index.md` — main entry point

### Navigation System
Navigation is entirely metadata-driven using `eleventyNavigation` frontmatter:

```yaml
eleventyNavigation:
  key: "Page Title"
  parent: "Parent Page Title"
  order: 1
```

- Pages under `src/docs/{plugin}/posts/*.md` are collected as a `posts` collection in `.eleventy.js`
- Sorted by `eleventyNavigation.order` (see `.eleventy.js` line 37)
- Use `parentFilter` to filter items by parent key (see `src/filters/parent-filter.js`)
- Breadcrumbs generated via `eleventyNavigationBreadcrumb` in `src/_includes/layouts/post.html`

### Template Hierarchy
1. **Base layout**: `src/_includes/layouts/base.html` — site wrapper, head, scripts
2. **Page layouts**:
   - `post.html` — documentation pages with TOC and breadcrumbs
   - `front-page.html` — plugin overview pages with buttons/FAQs
   - `simple.html` — minimal pages

### Custom Eleventy Extensions

**Filters** (`src/filters/`):
- `parentFilter(collection, parent)` — filter navigation items by parent key
- `dateFilter`, `w3DateFilter` — format dates for display/SEO

**Shortcodes** (`src/shortcodes/`):
- `{% svgIcon 'path/to/icon.svg', 'css-class' %}` — inlines SVG with custom class
- `{% markdownRender %}` — renders markdown content

**Transforms**:
- `html-min-transform.js` — minifies HTML in production (enabled via `NODE_ENV=production`)

### Data Files (`src/_data/`)
- `site.json` — site metadata (name, URL, social links)
- `global.js` — utilities like `random()` (UUID) and `year()` for copyright
- `helpers.js` — `getLinkActiveState()` for active navigation states
- `navigation.json` — global navigation structure

### Styling (Sass + SpruceCss)
- Entry point: `src/scss/main.scss` — forwards config/layout/section/component partials
- Uses **SpruceCss framework** (`@use 'sprucecss/scss/spruce'`)
- CSS variables defined in `:root` (see `main.scss`)
- Compiled via `npm run sass:dev` (watch) or `npm run sass:prod` (compressed)

## Development Workflow

### Commands
- **Start dev server**: `npm start` — runs Eleventy with live reload + Sass watch
- **Build production**: `npm run prod` — deletes dist/, builds minified site
- **Lint Sass**: `npm run sass:lint` / `npm run sass:lint:fix`
- **Lint JS**: `npm run js:lint` / `npm run js:lint:fix`

### Critical: Always Compile After Changes
**After making any changes to the codebase, you MUST compile:**
- **Sass changes**: Run `npm run sass:prod` or `npm run sass:dev` after editing any `.scss` files
- **Content/template changes**: Run `npm run eleventy:prod` or restart `npm start` for changes to `.md`, `.html`, `.js` data/filter/shortcode files
- **Complete rebuild**: Run `npm run prod` to compile everything and prepare for production

### Build Process
1. Sass compiles `src/scss/**` → `src/css/` (passthrough copied to dist)
2. Eleventy processes `src/` → `dist/`
3. Static assets (img, js, font, robots.txt) copied via `addPassthroughCopy`
4. Production builds minify HTML via transform

### Adding New Documentation
1. Create `src/docs/{plugin-name}/posts/{slug}.md`
2. Add frontmatter with `eleventyNavigation` (key, parent, order)
3. Set `layout: "layouts/post.html"` and `permalink: /docs/{plugin}/{slug}/`
4. Content auto-generates TOC if headings exist (see `eleventy-plugin-toc`)

## Key Patterns

### Markdown Configuration
Uses `markdown-it` with `markdown-it-anchor` for heading anchors:
- Permalink symbol: `#`
- Permalink class: `anchor`
- HTML allowed in markdown (`html: true`)

### Collections
The `posts` collection filters by glob pattern `./src/posts/**/posts/*.md` and sorts by `eleventyNavigation.order`. When adding plugin docs, ensure files match this pattern.

### Asset Handling
- **Images**: Place in `src/img/`, reference as `/img/file.png`
- **Fonts**: `src/font/` → copied to `/font/`
- **JavaScript**: `src/js/` → raw files copied (no bundling)
- **CSS**: Compiled Sass output in `src/css/` → copied to dist

### Disabled Features
- Pagefind search integration is commented out in `.eleventy.js` (binary download issues)

## WordPress Plugin Context
This site documents the **WebHotelier for WordPress** plugin (search availability forms for hotels using WebHotelier booking engine). When editing docs:
- Shortcode format: `[wp-webhotelier arg="value"]`
- Key arguments: `url`, `max-adults`, `orientation`, `checkout-date`, `opening-closing-dates`
- Three integration methods: shortcode, widget, Gutenberg block

## Conventions
- Use Nunjucks syntax (`{% %}`, `{{ }}`) in HTML templates
- Use YAML frontmatter in markdown files
- Keep navigation order sequential (1, 2, 3...)
- Always set `permalink` in frontmatter to control URLs
- Reference site data via `{{ site.name }}`, `{{ site.url }}`, etc.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Sotiris-k) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
