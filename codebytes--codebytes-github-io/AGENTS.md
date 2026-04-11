# Project standards

## Git Workflow
- Never commit directly to the `main` branch
- Always create a feature branch for changes
- Open a pull request to merge into `main`
- Use descriptive branch names (e.g., `feature/add-blog-post`, `fix/linting-errors`)

## Hugo & Blowfish Theme
- This site uses Hugo with the Blowfish theme (submodule in `themes/blowfish`)
- Hugo version must stay within the range declared in `themes/blowfish/config.toml` (`[module.hugoVersion]` min/max)
- Before upgrading Hugo, check the Blowfish `config.toml` `max` version constraint
- Update `HUGO_VERSION` in `.github/workflows/hugo.yml` to match any local Hugo version changes
- If Blowfish warns "is not compatible with this Hugo version", downgrade Hugo to the declared `max`

## Markdown Guidelines
- avoid special characters
- use headings to structure content
- use bullet points for lists
- use code blocks for code snippets
- use links for references
- use images with alt text
- use tables for data representation
- use blockquotes for quotes
- use horizontal rules for separation
- use footnotes for citations

## YAML Front Matter
- use `---` to start and end the front matter
- use `title`, `date`, `tags`, `categories`, `permalink`, `header`, and `excerpt_separator` as keys
- use strings for values
- use arrays for `tags` and `categories`
- use strings for `permalink`, `header`, and `excerpt_separator`
- use strings for `teaser` and `og_image` in the header
- use `<!--more-->` for content separation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/codebytes)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/codebytes)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
