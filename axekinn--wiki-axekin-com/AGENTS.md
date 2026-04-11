# .github/copilot-instructions.md

Purpose: Quickly orient an AI coding agent to be productive in this repository (Hexo static site using the Reimu theme).

- Project type: Hexo (see `package.json` -> `hexo.version: 8.1.1`). Build scripts are in `package.json`.
- Quick start commands:
  - Install deps: prefer `pnpm install` (there is a `pnpm-lock.yaml`) or `npm install` when pnpm is not available.
  - Build static site: `pnpm build` or `npm run build` (runs `hexo generate`).
  - Serve locally: `pnpm server` or `npm run server` (runs `hexo server`, default localhost:4000).
  - Clean generated files: `pnpm clean` or `npm run clean`.

- Key locations to read before editing:
  - `source/` — main site content; `source/_posts/` holds posts; localized content under `source/<lang>/`.
  - `themes/reimu/` — theme code, templates, assets, and theme-specific `_config.yml`.
  - `themes/reimu/_config.yml` — primary place for theme appearance, CSS variables (e.g. `internal_theme`), vendor CDN config, and feature toggles.
  - `scaffolds/` — front-matter templates for new pages/posts.
  - `db.json` — Hexo generated DB (read-only for source-of-truth during runtime).

- Important project patterns / conventions:
  - Theme variables: the theme defines CSS variables under `internal_theme: light` and `dark`. When adding colors, update both light and dark sections and reuse keys like `--red-0` / `--highlight-purple` to remain consistent.
  - Localization: translations are implemented by duplicating content under `source/<lang>/_posts/` and `source/<lang>/about/index.md`. Maintain parallel structure for each locale.
  - Asset paths in theme configs are relative to `themes/reimu/source/` or to the compiled CSS (see comments in `_config.yml`). Use existing image folders in `themes/reimu/source/images/`.
  - Feature toggles live in theme `_config.yml` (comments indicate dependencies/plugins). If enabling a feature, verify the required Hexo plugin is present in `package.json`.

- External integrations / points of attention:
  - Comments providers (waline, valine, giscus, etc.) are configured in `themes/reimu/_config.yml` — toggles + provider-specific keys exist there.
  - Vendor CDNs and JS/CSS fingerprints are configured under the `vendor` section in the theme config. Keep integrity hashes in sync if versions are changed.

- Debugging & developer workflow tips:
  - For theme/template changes, run `pnpm server` (or `npm run server`) and inspect browser console for missing assets or template errors; hexo regenerate/restart after changing layout files.
  - To preview only theme changes without altering content, edit files in `themes/reimu/` and reload the local server.
  - Deployment: `pnpm deploy` or `npm run deploy` (inspect repository-specific deploy settings outside this repo if needed).

- AI-specific notes and guidance:
  - Always scan `themes/reimu/_config.yml` when working on visual changes (colors, layout toggles, integrations). Search for `internal_theme` to find the CSS variables block.
  - When changing content rules (front-matter, toc, cover), check `scaffolds/` and existing post front-matter under `source/_posts/` for canonical examples.
  - Preserve localization file structure when adding or moving content — update each `source/<lang>/` copy appropriately.
  - Avoid changing `hexo` major version in `package.json` without testing locally; theme may rely on specific Hexo behavior.

- Examples to act on:
  - Change banner image: edit `themes/reimu/_config.yml` key `banner` (path under `themes/reimu/source/images/`).
  - Update theme color tokens: edit `internal_theme` entries in `themes/reimu/_config.yml` and keep both `light` and `dark` sections in sync.

If any section seems incomplete or you want examples expanded (deploy details, theme variable conventions, or localization process), tell me which area to expand.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Axekinn)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Axekinn)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
