# Copilot Instructions for Eldraeverse

This is a Ghost theme based on the Source theme, maintained for the Associated Worlds blog.

## Build, Test, and Lint Commands

- **`npm run dev`** or **`yarn dev`** - Start development mode with live reload
  - Watches `.hbs` files, `assets/css/**`, and `assets/js/**` for changes
  - Automatically rebuilds CSS and JS, triggers browser reload
  
- **`npm test`** - Build and validate the theme
  - Runs `gulp build` (pretest hook) to compile CSS and JS to `assets/built/`
  - Then runs `gscan` to check Ghost theme compatibility

- **`npm run zip`** or **`yarn zip`** - Create distributable `.zip` file
  - Runs build first, then packages theme (excludes node_modules, dist, gulpfile.js)
  - Output: `dist/eldraeverse.zip`

- **`npm run test:ci`** or **`yarn test:ci`** - Strict CI version of theme validation
  - Runs build first, then runs `gscan` with `--fatal` and `--verbose` flags
  - Treats warnings as errors (recommended for pre-deployment)

## Architecture

**Eldraeverse is a Ghost theme with the following structure:**

### Template Files (root `.hbs` files)
- **`home.hbs`** - Homepage (overrides default landing page)
  - Uses unified container layout with full sidebar on right
  - Sections (top to bottom): Latest (full post) → CTA → Featured → Other Recent Posts
  - All content in single `gh-main` column alongside sidebar
- **`index.hbs`** - Main index/archive page (full-width, no sidebar)
- **`post.hbs`** - Single post template
- **`page.hbs`** - Static page template
- **`page-random.hbs`** - Intermediate redirect page for `/random/` route
- **`tag.hbs`** - Tag archive page (full-width, no sidebar)
- **`author.hbs`** - Author profile page (full-width, no sidebar)
- **`custom-tag-cloud.hbs`** - Custom page template that renders normal page content plus a tag cloud
- **`default.hbs`** - Base wrapper for all templates

### Partials (`partials/` directory)
- **`components/`** - Reusable component partials
  - **`latest-full-post.hbs`** - Displays most recent post with full content on homepage
    - Includes post title (H3), feature image, metadata, full content, and "View post & comment" link
    - Used in home.hbs with full gh-content styling and paragraph spacing
  - **`featured.hbs`** - Featured posts section (supports `skipContainer` parameter)
  - **`post-list.hbs`** - Post feed with list/grid layouts (supports `skipContainer` and `skipFirst` parameters)
  - **`cta.hbs`** - Call-to-action/newsletter signup (shown on home between Latest and Featured)
- **`typography/`** - Font and text styling partials
- **`icons/`** - Icon SVG partials
- Post card variants: `post-card.hbs`, `related.hbs`, `related-simple.hbs`, `related-two.hbs`
- **`email-subscription.hbs`** - Newsletter signup form
- **`feature-image.hbs`** - Featured image display logic
- **`lightbox.hbs`** - Image lightbox implementation
- **`search-toggle.hbs`** - Search UI toggle

### Assets (`assets/` directory)
- **`css/screen.css`** - Main stylesheet (PostCSS with nested imports)
- **`js/`** - JavaScript source files
  - `lib/` - Library/dependency files (loaded first)
  - Other `.js` files (concatenated after lib)
- **`built/`** - Compiled/minified output (gitignored during development)
- **`fonts/`** - Web fonts
- **`images/`** - Static images
- **`icons/`** - SVG icons for use in templates

### Build Pipeline
The Gulp build system processes:
1. **CSS** - PostCSS imports (easyimport), autoprefixer, and cssnano minification
2. **JS** - Library files first, then application code, concatenated into `source.js`, uglified
3. **HBS files** - Watched for changes, trigger live reload (no processing)

## Key Conventions

### Homepage Layout Architecture

The homepage uses a **unified container layout** where the page template (home.hbs) owns the layout structure once, and components render content without their own container wrappers.

**Layout structure:**
- `<section class="gh-container has-sidebar gh-outer">` - Main container with 16-column grid
  - `<main class="gh-main">` - Left column (12 cols) containing all content sections
    - `Latest` heading + `latest-full-post` component (full post display)
    - `cta` component (newsletter signup)
    - `featured` component (featured posts section)
    - `Other Recent Posts` heading + `post-list` component (feed with 12 posts)
  - `<aside class="gh-sidebar">` - Right column (4 cols) with subscription/support options

**Key parameters for components:**
- `skipContainer=true` - Component renders content without its own `gh-container` wrapper (used by featured and post-list)
- `skipFirst=true` - Post-list skips the most recent post (displayed separately in Latest section above)
- Post fetch limits adjusted when `skipFirst=true`: add 1 to limit to compensate for skipped post

**Pages without sidebar:**
- `index.hbs`, `tag.hbs`, `author.hbs` - Full-width layout, no sidebar (each wraps content in single `gh-container`)
- `post.hbs` - Single post display (not refactored, different layout)

**CSS grid spacing:**
- Direct children of `gh-main` get margin spacing: 64px between sections
- `.gh-feed` gets gap spacing: `var(--grid-gap)` (42px) between post cards
- Both `.gh-container .gh-feed` and `.gh-main > .gh-feed` rules ensure spacing works in all contexts

### CSS
- Uses **PostCSS** with `postcss-easy-import` for modular stylesheets
- Autoprefixer adds vendor prefixes automatically
- cssnano minifies the output (no separate minification step needed)
- Source maps are generated during build

### JavaScript
- Organized with `lib/` subdirectory for dependencies loaded first
- Files are concatenated in glob order into `assets/built/source.js`
- Uglified automatically during build
- Source maps are generated during build

### Handlebars Templates
- Ghost context helpers and variables are available in all templates
- Partials use `{{> name}}` syntax
- Custom theme configuration exposed in `@custom` context (see package.json `custom` field)

### Component Parameters

**featured.hbs:**
- `showFeatured` - Boolean to show/hide featured section
- `limit` - Number of featured posts to display (default: 4)
- `skipContainer` - When true, omits outer `gh-outer`/`gh-inner` divs (used on home.hbs)

**post-list.hbs:**
- `feed` - Feed type: "home", "index", "archive" (determines which posts to fetch and how many)
- `postFeedStyle` - Layout style: "list" or "grid" (controls CSS class and display)
- `showTitle` - Boolean to show/hide section title
- `showSidebar` - Boolean to render sidebar (only used on archive/tag/author pages)
- `skipContainer` - When true, omits outer `gh-container`/`gh-main` divs (used on home.hbs)
- `skipFirst` - When true, skips the most recent post from feed (used on home.hbs)

**Homepage post fetch limits (post-list.hbs, feed="home"):**
When `skipFirst=false` (default):
  - Highlight with featured: 16 posts (skip 5)
  - Highlight without featured: 22 posts (skip 11)
  - Magazine: 19 posts (skip 8)
  - Default: 12 posts (show all)

When `skipFirst=true` (incremented by 1 to compensate for skipped post):
  - Highlight with featured: 17 posts (skip 6)
  - Highlight without featured: 23 posts (skip 12)
  - Magazine: 20 posts (skip 9)
  - Default: 13 posts (skip 2)

This ensures 12 posts appear in the "Other Recent Posts" section while the latest post is displayed separately above.

### Ghost Theme Configuration
- Theme customization options defined in `package.json` under `config.custom`
- Options include navigation layout, fonts, colors, and feature toggles
- Image sizes configured under `config.image_sizes`
- Posts per page configured in `config.posts_per_page` (default: 12)

### Special Features

#### Random Post route (`/random/`)
- `assets/js/random-post.js` handles any click to `/random/` and auto-handles direct visits to `/random/`.
- Candidate pool is sourced from Ghost post sitemap files (`/sitemap.xml` → `sitemap-posts*`) for full-history coverage.
- Empty candidate behavior: redirect to homepage.
- `page-random.hbs` is intentionally minimal and now has light centered styling so the brief redirect flash is visually consistent.
- Operational usage: create a Ghost page with slug `random`, then add `/random/` in Ghost navigation.

#### Tag cloud page
- `custom-tag-cloud.hbs` is registered in `package.json` templates as **Tag cloud page**.
- The page renders authored page content first, then `partials/components/tag-cloud.hbs`.
- Tag cloud fetch: `{{#get "tags" include="count.posts" limit="all" order="count.posts desc" filter="visibility:public"}}`.
- Size buckets in markup classes:
  - `is-xl` for `count.posts >= 50`
  - `is-l` for `>= 25`
  - `is-m` for `>= 10`
  - `is-s` for `>= 5`
  - `is-xs` otherwise

#### Legacy numeric-slug / WP-era Discourse support
- `post.hbs` supports comment loading by topic ID when URL-based lookup fails for legacy migrated posts.
- If `window.DISCOURSE_TOPIC_ID` is present (typically injected via `codeinjection_head` for specific legacy posts), the theme sets `DiscourseEmbed.topicId` and deliberately omits `discourseEmbedUrl`.
- If `window.DISCOURSE_TOPIC_ID` is absent, the default URL-based mode is used: `DiscourseEmbed.discourseEmbedUrl = '{{url absolute="true"}}'`.

### Release Workflow
- Use `npm run ship` to version and push (runs tests automatically)
- Use `npm run release` to create GitHub release draft (requires `GST_TOKEN` environment variable)
- Requires clean git working directory (no uncommitted changes)

### Compatibility
- Minimum Ghost version: **5.0.0** (defined in package.json engines)
- Currently tested and deployed against: **Ghost 6.32.0**
- Uses browserslist default targets for browser support
- Can use any features supported by Ghost 6.32.0 and later

## Deployment

**Target:** `wraith:/opt/ghost/data/ghost/themes/eldraeverse`

The theme is deployed using a tar archive to avoid syncing build artifacts and dependencies. Before deploying, ensure:
1. All changes are committed and pushed
2. Run `npm test` to validate the theme with gscan
3. Verify `assets/built/` contains the latest compiled CSS and JS (run `npm run build` if needed)

**Quick deployment:**
Use the provided deployment script to avoid manual command construction:
```bash
./_local/deploy.sh
```

**Manual deployment (if script unavailable):**

1. **Create a deployment archive** (excludes node_modules, build tools, sourcemaps):
```bash
tar --exclude=node_modules --exclude=.git --exclude=gulpfile.js --exclude=package-lock.json --exclude='*.map' --exclude='_local' -czf /tmp/eldraeverse-deploy.tar.gz .
```

2. **Transfer archive to wraith:**
```bash
scp /tmp/eldraeverse-deploy.tar.gz wraith:/tmp/
```

3. **Extract on wraith** (creates/updates the theme directory):
```bash
ssh wraith "mkdir -p /opt/ghost/data/ghost/themes/eldraeverse && cd /opt/ghost/data/ghost/themes/eldraeverse && tar -xzf /tmp/eldraeverse-deploy.tar.gz --strip-components=1"
```

4. **Restart Ghost** to reload the theme:
```bash
ssh wraith "cd /opt/ghost && docker compose up -d --force-recreate ghost caddy"
```

**What gets deployed:**
- All `.hbs` template and partial files
- Compiled `assets/built/screen.css` and `assets/built/source.js`
- Asset files (fonts, images, icons)
- `package.json` and metadata files

**What is excluded (not needed on production):**
- `node_modules/` — build tools only
- `.git/` — version control
- `gulpfile.js` — build configuration
- `*.map` — sourcemaps for debugging
- `package-lock.json` — lock file
- `_local/` — local development utilities (not deployed)

## Security

The theme includes the following security hardening measures:

### URL Validation
- **Pagination** - Validates all next-page URLs are same-origin before fetching, preventing open redirect XSS
- **Lightbox** - Validates image URLs use http/https protocol only, preventing data: URI and javascript: protocol attacks

### Input Validation
- **Color settings** - CSS custom color setting validated to hex format (#RRGGBB or #RGB), preventing CSS injection
- **Discourse configuration** - Removed hardcoded URLs; configured via Ghost Code Injection for security and portability

### DOM Safety
- **Dropdown menu** - Replaced innerHTML usage with safe DOM methods (createElement/createElementNS), eliminating HTML injection risk
- **SVG generation** - Uses createElementNS() for proper namespace handling instead of HTML parsing
- **Pagination** - Uses DOMPurify to sanitize fetched HTML before DOM insertion, preventing XSS from compromised pages

### HTML Sanitization
- **DOMPurify library** - Lightweight (~4KB gzipped) HTML sanitization library prevents XSS attacks
- **Pagination integration** - Fetched page HTML is sanitized before parsing, while pagination metadata is preserved
- **How it works**: Extract pagination link from raw HTML → Sanitize content with DOMPurify → Parse sanitized HTML → Use pre-extracted URL
- **Extensible**: DOMPurify can be applied to any other content source that needs sanitization

### Additional Recommendations

For defense-in-depth protection, configure a Content Security Policy (CSP) header at the Ghost/reverse proxy level. See `_local/CSP_SETUP.md` for detailed configuration guidance.

---
> Source: [cerebrate/Eldraeverse](https://github.com/cerebrate/Eldraeverse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-12 -->
