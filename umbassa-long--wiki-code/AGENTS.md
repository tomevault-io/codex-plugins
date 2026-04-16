# AI Coding Agent Instructions for Wiki-Code Knowledge Base

## Project Overview
This is a German-language knowledge base built with MkDocs and Material theme. It serves as a personal wiki for code snippets, how-tos, and project documentation.

## Architecture
- **Source**: `docs/` directory contains Markdown files
- **Build Output**: `site/` directory (generated)
- **Assets**: `docs/assets/` for CSS, `docs/bilder/` for images
- **Includes**: `includes/` for MkDocs macros (currently empty)

## Development Workflow
- **Install Dependencies**: `pip install -r requirements.txt`
- **Local Development**: `python -m mkdocs serve` (starts dev server with live reload)
- **Build Site**: `python -m mkdocs build --site-dir site --clean`
- **Deploy**: Push to GitHub main branch (deploys via GitHub Pages)

## Content Conventions
- **Language**: All content must be in German
- **Navigation**: Update `mkdocs.yml` nav section when adding new pages
- **Markdown Features**:
  - Use admonitions: `!!! note "Title"` for callouts
  - Emojis: `:material-icon:` syntax (e.g., `:material-code-tags:`)
  - Tables: Use pipe syntax with alignment
  - Code blocks: Specify language for syntax highlighting
- **File Naming**: Use kebab-case for filenames (e.g., `linux-commands.md`)

## Key Files
- `mkdocs.yml`: Site config, theme, plugins, navigation
- `requirements.txt`: Python dependencies (MkDocs, Material theme, plugins)
- `docs/index.md`: Homepage with overview and quick links

## Patterns to Follow
- **Page Structure**: Start with frontmatter if needed (e.g., `--- hide: - toc ---`)
- **Links**: Use relative paths for internal links
- **Images**: Place in `docs/bilder/` and reference as `../bilder/image.png`
- **Macros**: Use `{{ macros }}` syntax if adding dynamic content (via mkdocs-macros-plugin)

## Deployment
Site deploys to `https://wiki-code.github.io/` via GitHub Pages. Ensure builds pass before pushing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/umbassa-long) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
