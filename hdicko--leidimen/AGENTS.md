# Copilot Instructions - Leidimen Hugo Site

## Project Overview

This is a Hugo-based static site for **Leidimen**, a French solidarity association supporting villages in the Douentza region of Mali. The site features multilingual content, Netlify CMS integration, and custom taxonomies for organizing content by villages, moods, categories, and tags.

**Tech Stack:**

- Hugo 0.152.1 (unified local & Netlify version via hugo-installer)
- Bootstrap 5.3.8 + Bootstrap Icons 1.13.1
- Dart Sass 1.93.2 for SCSS processing
- PhotoSwipe for image galleries
- Netlify CMS for content management
- GitHub Pages + Netlify (dual deployment)
- Prettier with go-template plugin for code formatting

## Architecture & Key Concepts

### Custom Taxonomies (Critical!)

Hugo is configured with **4 custom taxonomies** (see `hugo.toml` line 73-76):

- `tags`: General keywords
- `categories`: Content classification (Éducation, Santé, Infrastructure)
- `moods`: Emotional tone (Heureux, Triste, Inspiré, Motivé, Reconnaissant)
- `villages`: Geographic classification (Douentza, Dorool, Diona, Debere, Diambana, Darawal, Tanal, Manko, Tacouti, N'Dumpa)

**When creating content:** Always use lowercase for village taxonomy values (e.g., `villages: ["dorool"]` not `"Dorool"`). Village data is centralized in `data/villages/mali_villages.yaml` with coordinates and project information.

### Content Structure

- **Posts**: `content/posts/` - **Organized by YEAR** (e.g., `2024/`, `2025/`) as primary structure
  - Standard posts: `content/posts/2024/my-article.md`
  - Bundle posts with images: `content/posts/2024/my-gallery/index.md` (images as page resources)
  - Legacy: Some posts exist in topic directories (`Hammadoun/souvenir/`) from WordPress migration
  - **Convention**: New posts MUST go in year-based directories for consistency and maintainability
  - Use `index.md` for bundle pages with images as page resources
- **Team profiles**: `content/equipe/` - Member profiles with card metadata
  - Each member gets a file like `abdoullayedicko.md`
  - Requires frontmatter: `card: {image, name, title, member, description, social}`
  - Note: Directory is `equipe` (French for "team"), not `about`
- **Galleries**: `content/galleries/` - Photo collections using PhotoSwipe
  - Structure: `galleries/gallery-name/index.md` + images in same folder
  - Images MUST be page resources (not in `static/`) for gallery shortcode to work
- **Villages**: `content/villages/` - Village information pages
  - One directory per village: `villages/dorool/`, `villages/diona/`
  - Maps use coordinates from `data/villages/mali_villages.yaml`
- **Documents**: `content/documents/` - Legal docs (RGPD, mentions légales)
  - Special layout in `layouts/documents/single.html`

### Layout System

- **Base template**: `layouts/_default/baseof.html` - Master wrapper with navbar, footer, Netlify Identity
- **Specialized layouts**:
  - `layouts/equipe/single.html` - Team profiles with timeline, skills, AOS animations
  - `layouts/galleries/single.html` - PhotoSwipe integration
  - `layouts/posts/list.html.html` - Blog listing with pagination

### PhotoSwipe Pattern (Important!)

The gallery system uses **Page.Scratch** to prevent duplicate PhotoSwipe loading:

```go
{{- if not ($.Page.Scratch.Get "photoswipeloaded") -}}
{{ $.Page.Scratch.Set "photoswipeloaded" 1 }}
{{ partial "photoswipe-resources.html" . }}
{{ partial "photoswipe-structure.html" . }}
{{- end -}}
```

- `gallery` shortcode auto-loads PhotoSwipe
- `load-photoswipe` shortcode for manual loading
- Never load PhotoSwipe twice on the same page

### Image Processing Convention

Standard image processing uses Hugo's image pipeline:

```go
{{- $resized := .Fill "400x500 top webp q85 lanczos" -}}
```

- **Format**: WebP for compression (fallback to original format if WebP fails)
- **Quality**: 85 (balance between size and quality)
- **Algorithm**: Lanczos (high quality downscaling)
- **Anchor**: `top` for portraits, `center` for landscapes, `smart` for adaptive cropping

**Common sizing patterns:**

- Profile photos: `400x400 center`
- Gallery thumbnails: `400x500 top`
- Post covers: `1200x630` for social media
- Page resources: Use `.Fill` for exact dimensions, `.Resize` for max dimensions

## Development Workflows

### Local Development

```bash
# Start dev server (overrides baseURL for local)
./dev-server.sh
# OR
npm run dev
# Access at: http://localhost:1313
# Netlify CMS at: http://localhost:1313/leidimen/admin/
```

### Deployment

**Two deployment paths:**

1. **GitHub Pages**: `./deploy.sh` - Builds and force-pushes to `gh-pages` branch
2. **Netlify**: Auto-deploys from `main` branch via `netlify.toml`

**Critical:** The site uses `baseURL = 'https://hdicko.github.io/leidimen/'` with `relativeURLs = false` for GitHub Pages subdirectory hosting.

### Content Creation

**Three content creation methods:**

1. **Web CMS Interface (cms-web/)** - Custom Node.js app for GitHub API-based content creation

   ```bash
   cd cms-web
   npm install
   # Configure GITHUB_TOKEN in .env
   npm start  # Access at http://localhost:3000
   ```

   - Direct GitHub API commits (no Netlify Identity required)
   - Form-based post creation with validation
   - Real-time Markdown preview and statistics
   - Token setup: https://github.com/settings/tokens (needs `repo` permission)

2. **Netlify CMS** - Browser-based Git-backed CMS
   - Local: http://localhost:1313/leidimen/admin/
   - Production: https://hdicko.github.io/leidimen/admin/
   - Authentication: Netlify Identity (configured in `baseof.html`)
   - Config: `static/admin/config.yml`

3. **Hugo Archetypes** - Command-line content generation
   ```bash
   hugo new posts/2025/my-article.md  # Uses archetype/post.md template
   ```

**When to use each CMS:**

- **cms-web** (Node.js interface):
  - ✅ Best for: Developers with Node.js, need for real-time validation and Markdown preview
  - ✅ Fastest workflow for batch content creation
  - ✅ Direct GitHub API commits (no Netlify Identity setup needed)
  - ❌ Requires: Node.js installed, GitHub token configured, local environment
- **Netlify CMS** (Web interface):
  - ✅ Best for: Non-technical editors, production environment, image uploads
  - ✅ Drag-and-drop image uploads to `static/images/uploads/`
  - ✅ Works remotely via browser (no local setup)
  - ❌ Requires: Netlify Identity authentication, internet connection
- **Hugo CLI** (Archetypes):
  - ✅ Best for: Developers comfortable with CLI, quick drafts, automated scripts
  - ✅ Fastest for experienced Hugo users
  - ✅ Full control over frontmatter and file structure
  - ❌ Requires: Hugo installed, manual frontmatter editing

### Code Formatting

```bash
npm run format:check   # Check formatting without changes
npm run format:write   # Auto-format all files
```

- Prettier is configured with `prettier-plugin-go-template` for Hugo templates
- Formats `.html` files in `layouts/`, `.scss` in `assets/`, `.md` in `content/`
- Configuration in `.prettierrc` (if exists) or `package.json`
- **Always format before committing** to maintain consistency

### Migration Scripts

Python scripts for migrating legacy WordPress content (leidimen.com archives 2006-2017):

**Available scripts:**

- `migrate-wordpress-posts.py` - Migrate posts from WordPress archives by date
  - Scrapes posts from https://leidimen.com archives (2006-2017)
  - Converts HTML to Markdown with Hugo shortcodes
  - Creates posts in year-based directories: `content/posts/YYYY/slug.md`
  - Maps WordPress categories to Hugo taxonomies
  - Usage: `python3 migrate-wordpress-posts.py`

- `migrate-wordpress-galleries.py` - Migrate photo galleries from WordPress
  - Creates Hugo page bundles with `index.md` + images
  - Generates PhotoSwipe-compatible gallery structure
  - Output: `content/galleries/{slug}/index.md`
  - Usage: `python3 migrate-wordpress-galleries.py`

- `migrate-wordpress-categories.py` - Migrate posts by WordPress categories
  - Targets specific categories: Divers, News, Réunions, Sorties
  - Maps WordPress categories to Hugo taxonomies
  - Usage: `python3 migrate-wordpress-categories.py`

- `update-image-links.py` - Update WordPress image URLs to local paths
  - Converts `leidimen.com/wp-content/uploads/YYYY/MM/image.jpg` → `/images/wordpress/YYYY-MM-image.jpg`
  - Runs in-place updates on existing Markdown files
  - Usage: `python3 update-image-links.py`

- `download-all-wordpress-images.py` - Download images from WordPress URLs
  - Scans content files for WordPress image URLs
  - Downloads to `static/images/wordpress/`
  - Usage: `python3 download-all-wordpress-images.py`

**Migration workflow (for new WordPress imports):**

```bash
# 1. Migrate posts
python3 migrate-wordpress-posts.py

# 2. Update image links in content
python3 update-image-links.py

# 3. Download referenced images
python3 download-all-wordpress-images.py

# 4. Test build
npm run build
```

### Testing Procedures

Comprehensive testing script: `test-hugo-compatibility.sh`

**What it tests:**

1. **Hugo version verification** - Ensures Hugo 0.152.1 is installed
2. **Development build** - Tests `hugo server` starts without errors
3. **Production build** - Tests minification and optimization
4. **File generation** - Verifies critical files exist:
   - `public/index.html`
   - `public/posts/` with post pages
   - `public/css/` with stylesheets
5. **Minification** - Validates HTML/CSS compression
6. **Taxonomies** - Checks villages, categories, tags, moods directories
7. **Image processing** - Verifies Hugo image pipeline works
8. **Shortcodes** - Tests custom shortcode rendering
9. **RSS/JSON feeds** - Validates output formats
10. **Sitemap** - Checks sitemap.xml generation

**Running tests:**

```bash
# Full compatibility test suite
./test-hugo-compatibility.sh

# Quick validation workflow
npm run build              # Build site
npm run format:check       # Check code formatting
./dev-server.sh            # Manual browser testing
```

**Test before deploying:**

```bash
./test-hugo-compatibility.sh && ./deploy.sh
```

## Project-Specific Conventions

### Frontmatter Structure

**Posts** must include:

```yaml
---
title: "Article Title"
date: 2025-10-22
villages: ["dorool", "diona"] # lowercase!
categories: ["Éducation"]
tags: ["école", "projet"]
description: "SEO description (150-160 chars)"
image: "/images/uploads/cover.jpg"
draft: false
---
```

### Hugo Version Management

- **Local & Netlify**: Hugo 0.152.1 (unified version)
- Version pinned in `package.json` under `otherDependencies.hugo`
- Netlify version configured in `netlify.toml` (HUGO_VERSION=0.152.1)
- **Unified version eliminates compatibility issues**
- Hugo binary location: `node_modules/.bin/hugo/hugo` (auto-installed via `hugo-installer` on `npm install`)
- **Critical**: Always run `npm install` after cloning to install Hugo binary locally

**Running Hugo commands:**

```bash
# Use npm scripts (recommended - uses exec-bin wrapper)
npm run dev      # hugo server --disableFastRender --gc
npm run build    # hugo --gc --cleanDestinationDir --minify

# Or use the wrapper script (overrides baseURL for local dev)
./dev-server.sh

# Direct hugo binary (after npm install)
exec-bin node_modules/.bin/hugo/hugo server
```

### Netlify CMS Config

Located at `static/admin/config.yml`:

- Git Gateway backend (GitHub authentication)
- Local backend enabled for development
- Media folder: `static/images/uploads/`
- Collections: Posts, About, Galleries, Villages, Documents

### Custom Shortcodes

Key shortcodes in `layouts/shortcodes/`:

**Gallery & Images:**

- `{{< gallery >}}` - Auto photo gallery from page resources (auto-loads PhotoSwipe)
- `{{< gallery-pro >}}` - Professional gallery with advanced options
- `{{< image-gallery >}}` - Alternative image gallery implementation
- `{{< gallery-item >}}` - Individual gallery item (used within image-gallery)
- `{{< load-photoswipe >}}` - Manual PhotoSwipe loading (use BEFORE custom galleries)
- `{{< image >}}` - Responsive image with lazy loading
- `{{< myimage >}}` - Custom image shortcode variant

**Layout & UI:**

- `{{< divider >}}` - Horizontal separator
- `{{< details "Summary" >}}Content{{< /details >}}` - Collapsible content
- `{{< typeit >}}Animated text{{< /typeit >}}` - Typing animation

**Note:** Hugo provides built-in shortcodes (figure, youtube, etc.) that can be used without custom implementations.

## Integration Points

### External Dependencies

- **Bootstrap 5.3.8**: UI framework (loaded in partials/head.html)
- **Bootstrap Icons**: Icon set
- **PhotoSwipe**: Lightbox library for galleries
- **Netlify Identity**: CMS authentication
- **Dart Sass**: CSS preprocessing (installed in Netlify build)

### Data Files

Hugo data files are accessible in templates via `.Site.Data`:

- `data/villages/mali_villages.yaml` - Village coordinates, population, projects
  - Access: `{{ range .Site.Data.villages.mali_villages }}`
  - Used for: Maps, village pages, statistics
  - **Pattern**: Single source of truth for all village data (coordinates, population, projects)
  - **Why**: Centralizes geographic data rather than duplicating in frontmatter
- `data/membres/adherents.json` - Member data (if used)
  - Alternative to individual content files for bulk members
- `data/settings/general.yml` - Site-wide settings
  - Contact info, social links, site metadata
- `data/map/vacations_spots.yaml` - Map markers data

**Data-driven pattern**: Village taxonomy in frontmatter (`villages: ["dorool"]`) links to detailed data in `mali_villages.yaml`. Templates fetch coordinates/population from data file using village name as key.

### Asset Pipeline

- Raw assets: `assets/scss/`, `assets/js/`
- Processed output: `public/css/`, `public/js/`
- Static files: `static/` (copied as-is to `public/`)

## Common Tasks

### Adding a New Village

1. Add village data to `data/villages/mali_villages.yaml`:
   ```yaml
   - name: "NewVillage"
     latitude: 15.xxxx
     longitude: -3.xxxx
     type: "village"
     description: "Description"
     population: 1000
     projects: ["école", "santé"]
   ```
2. Create content page: `content/villages/newvillage/_index.md`
3. Use lowercase in posts: `villages: ["newvillage"]`
4. Update Netlify CMS config: `static/admin/config.yml` (add to villages options)

### Creating a Photo Gallery Post

1. Create bundle structure:
   ```
   content/posts/2025/my-gallery/
   ├── index.md
   ├── photo1.jpg
   ├── photo2.jpg
   └── photo3.jpg
   ```
2. In `index.md` frontmatter:

   ```yaml
   ---
   title: "My Gallery"
   date: 2025-10-22
   type: "posts"
   ---
   { { < gallery > } }
   ```

3. Images MUST be in the same folder (page resources), not in `static/`
4. Gallery shortcode auto-loads PhotoSwipe—no manual loading needed

### Adding a Team Member

1. Create file: `content/equipe/firstname-lastname.md`
2. Required frontmatter structure:
   ```yaml
   ---
   title: "Full Name"
   card:
     image: "/images/team/firstname.jpg"
     name: "Full Name"
     title: "Role Title"
     member: ["membre", "fondateur", "CA"]
     description: "Short bio"
     social:
       - { icon: "envelope", link: "email@example.com" }
       - { icon: "linkedin", link: "https://linkedin.com/..." }
   experience:
     - { year: "2020", title: "Position", company: "Organization" }
   skills: ["Skill 1", "Skill 2"]
   ---
   ```
3. Image goes to `static/images/team/` (not page resource)
4. Layout at `layouts/equipe/single.html` uses AOS animations and timeline components

### Troubleshooting Gallery Issues

- **Gallery not showing**: Check that images are page resources (in same folder as `index.md`)
- **PhotoSwipe loaded twice**: Verify only ONE of `{{< gallery >}}` or `{{< load-photoswipe >}}` is used
- **Images not processing**: Ensure file is named `index.md` not `my-post.md` for bundles
- **Lightbox not working**: Check browser console for JS errors, verify PhotoSwipe loaded

## Critical Gotchas

1. **Taxonomy values must be lowercase** in frontmatter (e.g., `villages: ["dorool"]` not `["Dorool"]`)
2. **PhotoSwipe loading must use Page.Scratch** to prevent duplicates—never call partials directly
3. **baseURL includes `/leidimen/` subdirectory**—use `.RelPermalink` not `.Permalink` for relative paths
4. **Hugo version unified at 0.152.1**—same version local and Netlify for consistency
5. **Content in `content/posts/` must use `type: "posts"`** in frontmatter for proper routing
6. **Gallery shortcode requires page resources**—images must be in same folder as `index.md`, not `static/`
7. **Netlify CMS media folder** is `static/images/uploads/`—these are NOT page resources
8. **Git quotepath false** required in Netlify build for French characters in filenames

## Key Files Reference

**Configuration:**

- `hugo.toml` - Site configuration, taxonomies, imaging settings, markup options
- `netlify.toml` - Netlify build config (Hugo 0.152.1, Dart Sass 1.93.2, build command)
- `package.json` - npm dependencies, scripts, Hugo version pinning
- `static/admin/config.yml` - Netlify CMS collections, fields, media settings

**Templates:**

- `layouts/_default/baseof.html` - Master wrapper with Netlify Identity integration
- `layouts/equipe/single.html` - Team profiles with AOS animations, timeline, skills
- `layouts/galleries/single.html` - Gallery pages with PhotoSwipe
- `layouts/posts/list.html.html` - Blog listing with pagination
- `layouts/partials/photoswipe-resources.html` - PhotoSwipe CSS/JS loader
- `layouts/partials/photoswipe-structure.html` - PhotoSwipe HTML structure

**Content Templates:**

- `archetypes/post.md` - Template for `hugo new posts/...`
- `archetypes/about.md` - Template for team member profiles (creates in equipe/)
- `archetypes/default.md` - Fallback template

**Scripts:**

- `deploy.sh` - Deploy to GitHub Pages (builds to `public/`, force-pushes to `gh-pages`)
- `dev-server.sh` - Local dev server with baseURL override
- `assets/js/darkmode.js` - Theme switcher
- `assets/js/profile.js` - Profile page interactions

**Documentation:**

- `QUICK_REFERENCE.md` - Content creation guide for non-technical editors
- `CODE_DOCUMENTATION.md` - Detailed code architecture documentation
- `CONTENT_CREATION_GUIDE.md` - Complete guide for content creation workflows
- `NETLIFY_CMS_GUIDE.md` - CMS usage instructions
- `SEARCH_FEATURE.md` - Search functionality documentation
- `SEO_LLM_OPTIMIZATION_SUMMARY.md` - SEO and AI optimization strategies
- `HUGO_0.152.1_MIGRATION.md` - Hugo version migration notes
- `cms-web/README.md` - Web CMS interface documentation with API integration
- `cms-web/QUICKSTART.md` - Quick start guide for cms-web
- `cms-web/INTEGRATION.md` - CMS integration details

## Testing & Validation

**Before committing:**

1. Run local dev server: `./dev-server.sh`
2. Check formatting: `npm run format:check`
3. Build site: `npm run build` (check for errors)
4. Verify taxonomy values are lowercase
5. Test galleries work (PhotoSwipe loads correctly)

**Common build errors:**

- **"can't find page resource"**: Image not in same folder as `index.md`
- **"taxonomy not found"**: Check taxonomy name in `hugo.toml` matches frontmatter
- **"template not found"**: Verify `type: "posts"` in frontmatter
- **SCSS compile error**: Check Dart Sass is installed/accessible
- **baseURL issues**: Verify `/leidimen/` path in production URLs

**Common build errors:**

- **"can't find page resource"**: Image not in same folder as `index.md`
- **"taxonomy not found"**: Check taxonomy name in `hugo.toml` matches frontmatter
- **"template not found"**: Verify `type: "posts"` in frontmatter
- **SCSS compile error**: Check Dart Sass is installed/accessible
- **baseURL issues**: Verify `/leidimen/` path in production URLs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hdicko)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/hdicko)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
