# Copilot Instructions

Keep responses concise and to the point.

This is a personal blog built with [Zola](https://www.getzola.org/) (v0.22.1), deployed to Netlify.

## Commands

```bash
zola serve           # Local dev server at http://127.0.0.1:1111
zola serve --drafts  # Include draft posts
zola build           # Build to _site/
zola check           # Check internal links
./new-post.sh        # Create a new post (prompts for title, sets draft: true)
```

## Architecture

- **`content/blog/`** — Markdown posts; filenames follow `YYYY-MM-DD-slug.md`
- **`templates/`** — Tera templates; `base.html` is the root layout
  - `includes/` — Partials: navigation, footer, post meta (dates, taxonomy)
  - `macros/macros.html` — `title()` macro used by all page templates
  - `blog-page.html` — Single post; `index.html` — Paginated post list
- **`sass/`** — SCSS compiled by Zola; `main.scss` imports all partials via `@use`
- **`static/`** — Copied verbatim; images go in `static/images/post-images/YYYY/`
- **`config.toml`** — Site config; nav menu items defined under `[extra].menus`

## Post Front Matter

```toml
+++
title = "Post Title"
date = 2026-01-06
slug = "2026-01-06-post-title"
taxonomies.categories = ["Category"]
taxonomies.tags = ["Tag One", "Tag Two"]
draft = false
+++
```

The `slug` field must be set explicitly (matching the filename) for correct pagination order.  
Use `<!--more-->` to set the summary shown on the home page listing.

## Key Conventions

**Internal links** use Zola's `@` syntax:
```markdown
[link text](@/blog/2023-09-06-selecting-a-post-processor.md)
```

**Images** use raw HTML (Zola has no image shortcode). Standard patterns from `zola_notes.md`:
```html
<!-- Centered -->
<img src="/images/post-images/2026/file.jpg" alt="..." style="display: block; margin: 1em auto 1em auto">

<!-- Float right -->
<img src="/images/post-images/2026/file.jpg" alt="..." style="float: right; width: 50%; margin: 1em 0em 1em 1em">

<!-- Figure with caption -->
<figure style="float: right; width: 50%; margin: 1em 0em 1em 1em">
  <img src="/images/post-images/2026/file.jpg" alt="...">
  <figcaption>Caption text</figcaption>
</figure>
```

**CSS** uses the [CUBE CSS](https://cube.fyi/) methodology with Utopia fluid type/space scales. Design tokens are CSS custom properties in `_global.scss`. Dark theme is applied via a `.dark` class on `<html>`, toggled by `localStorage["color-theme"]`.

**Analytics** (Woopra) only loads when the `env` environment variable is `prod` — it is skipped during local development.

**Taxonomy** requires both `categories` and `tags` keys to be present in front matter (can be empty arrays).

---
> Source: [bobrocke/My-Blog-Zola](https://github.com/bobrocke/My-Blog-Zola) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-22 -->
