# Copilot Instructions for charleshancoder.github.io

## Project Overview
This is a Jekyll-based static site, primarily used for technical blogging, documentation, and knowledge sharing. Content is organized in Markdown files and rendered using Jekyll layouts and includes. The site covers topics such as API management, .NET migration, infrastructure as code, and developer tools.

## Key Directories & Files
- `_posts/`: Blog posts in Markdown, named with date prefixes (e.g., `2024-08-01-dotnet-migration.md`).
- `_drafts/`: Unpublished or in-progress articles.
- `_layouts/`, `_includes/`: Jekyll HTML templates for page rendering.
- `_data/`: YAML files for navigation and quotes.
- `assets/`: Static assets (CSS, fonts, images, JS).
- `Gemfile`, `Gemfile.lock`: Ruby dependencies for Jekyll.
- `_config.yml`: Jekyll site configuration.
- `.github/`: Place for workflow and agent instructions.

## Developer Workflows
- **Local Development**: Use `bundle exec jekyll serve` to build and serve the site locally. Default port is 4000.
- **Content Editing**: Write posts in Markdown. Use YAML front matter for metadata (title, date, layout).
- **Site Build**: Jekyll automatically processes Markdown and templates into HTML in `_site/`.
- **Deployment**: Push changes to the `master` branch; GitHub Pages will auto-deploy.
- **Dependencies**: Manage Ruby gems via `bundle install`.

## Project-Specific Patterns
- **Date-based Post Naming**: All posts in `_posts/` use `YYYY-MM-DD-title.md` format for chronological sorting.
- **Custom Layouts**: Use `layout: default` or `layout: post` in front matter to select templates.
- **Navigation**: Controlled via `_data/navigation.yml`.
- **Quotes**: Managed in `_data/quotes.yml` and rendered on pages with `show_quote: true`.
- **Math & Code**: Use standard Markdown and LaTeX for math (`$A = \pi r^2$`).
- **Extensions**: Documented in `tools.md` for VS Code and Visual Studio.

## Integration Points
- **GitHub Pages**: Automatic deployment from `master` branch.
- **Jekyll Plugins**: Managed via `Gemfile`.
- **External Libraries**: Referenced in `libs/` and documented in `libs.md`.

## Conventions & Tips
- Avoid editing `_site/` directly; it is auto-generated.
- Use drafts for incomplete articles; move to `_posts/` when ready.
- YAML front matter is required for all Markdown content.
- For new pages, add navigation entries in `_data/navigation.yml`.
- Use relative paths for images and assets.

## Example: Adding a New Blog Post
1. Create `YYYY-MM-DD-title.md` in `_posts/`.
2. Add YAML front matter:
   ```yaml
   ---
   title: "My New Post"
   date: 2025-07-19
   layout: post
   ---
   ```
3. Write content in Markdown.
4. Commit and push to `master`.

---
For questions or unclear conventions, review `_config.yml`, `tools.md`, and existing posts for examples. Iterate and ask for feedback if unsure about structure or workflow.

---
> Source: [charleshancoder/charleshancoder.github.io](https://github.com/charleshancoder/charleshancoder.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-23 -->
