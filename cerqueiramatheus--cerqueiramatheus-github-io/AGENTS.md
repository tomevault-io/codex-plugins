# Copilot Instructions — mhcp.dev

## Commands

```sh
npm run dev            # Convert .tex then start dev server
npm run build          # Convert .tex then production build (output: build/)
npm run check          # TypeScript + Svelte type-checking
npm test               # Run tests
npm run test:coverage  # Run tests with coverage report
npm run new            # Scaffold a new post or presentation
./run.sh               # Install, build, and preview
```

Requires `pandoc` installed locally and on CI (pre-installed on GitHub Actions `ubuntu-latest`).

## Architecture

SvelteKit static site (`@sveltejs/adapter-static`) deployed to GitHub Pages at `www.mhcp.dev`. Extreme minimalist design: Cascadia Mono font, two-color monochrome palette, no decorative elements.

**Build pipeline:**
1. `scripts/tex2html.mjs` runs Pandoc to convert `.tex` → `.html` into `.generated/`
2. Vite builds the SvelteKit app, loading generated HTML via `import.meta.glob(?raw)`
3. `adapter-static` outputs the final site to `build/`

**Configuration:**
- `config.yml` (root) — All site strings, URLs, profile, social links, nav, font, AdSense. Parsed by `js-yaml` at build time.
- `src/lib/config.ts` — Typed re-export of `config.yml`.

**Content:**
```
content/
  posts/<slug>/
    article.tex        # LaTeX source (metadata in % comments: title, date, description)
    refs.bib           # Per-post BibTeX references (optional)
    assets/            # Images, SVGs (auto-copied to static/posts/<slug>/)
  presentations/<slug>/
    article.tex        # LaTeX source (metadata includes % url: for PDF embed)
  pages/
    about.tex          # Standalone page (LaTeX)
  refs.bib             # Site-wide publications (rendered on /publications)
  ieee.csl             # Citation style for Pandoc citeproc
```

**Route structure:**
- `/` — Landing: avatar, name, role, social text links. Vertically centered.
- `/about` — About page (LaTeX → HTML)
- `/posts` — Post list with search, year filter, pagination
- `/posts/[slug]` — Post detail (LaTeX → HTML via Pandoc, with MathML + citeproc)
- `/presentations` — Presentation list with search, year filter, pagination
- `/presentations/[slug]` — Presentation detail with embedded PDF iframe + fullscreen
- `/publications` — Publications from `content/refs.bib` in IEEE format

## Key conventions

- **Svelte 5 runes**: `$props()`, `$derived`. No legacy syntax.
- **Extreme minimalism**: No borders, shadows, radius, gradients, icons, emoji, favicons, transitions. Links always underlined. All titles lowercased via CSS `text-transform`.
- **Cascadia Mono**: Single monospace font. Loaded via `@font-face` in `app.css`.
- **Two colors**: `#111` text, `#fafafa` background.
- **LaTeX content**: All content is `.tex`. Math uses `$...$` and `$$...$$` — Pandoc converts to MathML (zero client JS). Code uses `\begin{verbatim}`.
- **Citations**: Per-post `refs.bib` + `\cite{}` in `.tex`. Pandoc `--citeproc --csl ieee.csl` renders IEEE-format references.
- **Adding a post**: Create `content/posts/<slug>/` with `article.tex` (and optionally `refs.bib`, `assets/`). Or run `npm run new`.
- **Adding a presentation**: Create `content/presentations/<slug>/` with `article.tex` including `% url:` for the PDF embed. Or run `npm run new`.
- **AdSense**: Library script in `app.html` `<head>` (enables ad serving on subdomains).
- **Deploy**: Push to `main` triggers `.github/workflows/deploy.yml`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CerqueiraMatheus) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
