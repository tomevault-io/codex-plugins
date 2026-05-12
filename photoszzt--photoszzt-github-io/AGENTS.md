# Copilot Instructions for `al-folio`

## Overview

`al-folio` is a Jekyll-based theme designed for academic websites. It provides a clean, responsive design with features like publications, CVs, and project showcases. The project is structured to support customization and deployment workflows.

### Key Components

- **Layouts**: Found in `_layouts/`, these define the structure of pages (e.g., `default.liquid`, `post.liquid`).
- **Includes**: Reusable components in `_includes/` (e.g., `header.liquid`, `footer.liquid`).
- **Data Files**: YAML files in `_data/` (e.g., `citations.yml`, `repositories.yml`) store structured data for dynamic content.
- **Assets**: Static files like CSS and JavaScript are in `assets/` and `_sass/`.
- **Plugins**: Ruby scripts in `_plugins/` extend Jekyll functionality (e.g., `cache-bust.rb`, `google-scholar-citations.rb`).
- **Posts and Pages**: Markdown files in `_posts/` and `_pages/` define site content.

## Developer Workflows

### Local Development

1. Install dependencies:
   ```bash
   bundle install
   npm install
   ```
2. Start the development server:
   ```bash
   bundle exec jekyll serve
   ```
   Access the site at `http://localhost:4000`.

### Deployment

- Deployment is automated via GitHub Actions (`.github/workflows/deploy.yml`).
- Ensure changes are committed to the `main` branch for deployment.

### Testing

- Use `lychee` for link checking:
  ```bash
  lychee --verbose ./
  ```
- Run Prettier for code formatting:
  ```bash
  npx prettier --check .
  ```

## Project-Specific Conventions

### Content Organization

- **Posts**: Use `_posts/` for blog entries. Follow the naming convention `YYYY-MM-DD-title.md`.
- **Pages**: Use `_pages/` for static pages (e.g., `about.md`, `projects.md`).
- **Publications**: Manage bibliography in `_bibliography/papers.bib`.

### Styling

- Customize themes in `_sass/_themes.scss`.
- Define global variables in `_sass/_variables.scss`.

### Data-Driven Content

- Use `_data/` for structured data (e.g., `repositories.yml` for GitHub stats).
- Reference data files in Liquid templates using `site.data`.

## Integration Points

### External Dependencies

- **Jekyll Plugins**: Listed in `Gemfile` (e.g., `jekyll-scholar`, `jekyll-feed`).
- **Node.js Packages**: Listed in `package.json` (e.g., `prettier`, `purgecss`).

### Cross-Component Communication

- Use Liquid includes (`{% include %}`) to share components across layouts and pages.
- Data files (`_data/`) are accessible globally in templates.

## Examples

### Adding a New Blog Post

1. Create a new file in `_posts/`:

   ```markdown
   ---
   layout: post
   title: "My New Post"
   date: 2026-01-04
   tags: [example, blog]
   ---

   Content goes here.
   ```

2. Preview locally using `jekyll serve`.

### Customizing the CV Page

1. Update `_data/cv.yml` with your details.
2. Modify `_layouts/cv.liquid` for layout changes.

---

For more details, refer to the [README.md](../README.md) and [INSTALL.md](../INSTALL.md).

---
> Source: [photoszzt/photoszzt.github.io](https://github.com/photoszzt/photoszzt.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-11 -->
