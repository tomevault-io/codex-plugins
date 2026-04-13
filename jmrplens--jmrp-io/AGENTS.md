# GitHub Copilot Instructions — jmrp.io

> For full project context, see [`CLAUDE.md`](../CLAUDE.md) at the project root.

## Coding Conventions

### TypeScript

- Strict mode (`astro/tsconfigs/strict`)
- Use path aliases: `@components/*`, `@layouts/*`, `@utils/*`, `@assets/*`, `@styles/*`, `@data/*`, `@languages/*`, `@src/*`, `@i18n/*`
- Prefer `interface` over `type` for component Props
- Use JSDoc comments for all exported functions and interfaces
- No `any` — use `unknown` with type guards

### Astro Components

- Use `.astro` for all UI — no `.tsx`/`.jsx` except Preact islands in `src/components/homelab/`
- Props interface at top of frontmatter: `interface Props { ... }`
- CSS scoped with `<style>` block (no `is:global` unless justified)
- Use UnoCSS utility classes, not inline styles
- Icons: `<span class="i-{collection}:{name}"></span>` (12 collections: mdi, logos, simple-icons, devicon, carbon, tabler, heroicons, lucide, fa-solid, fa-brands, fa-regular, vscode-icons)

### CSS & Styling

- Dark-first theme — dark mode is default, light is the override
- Use CSS custom properties (`--color-*`, `--space-*`, `--font-*`) from `src/styles/global.css`
- WCAG AA contrast minimum (4.5:1 for text, 3:1 for large text)
- No fixed pixel widths — use `%`, `rem`, `ch`
- No `!important` — fix specificity instead
- Stylelint enforces kebab-case BEM selectors

### MDX Blog Posts

- File naming: `NNN-slug.mdx` (e.g., `009-post-name.mdx`)
- `description` ≤ 155 chars (enforced by tests)
- Import components from `@components/ui/` — see `src/components/ui/AGENTS.md`
- `<Mermaid>` requires `ariaLabel` prop
- All images require descriptive `alt` text
- Heading hierarchy: h1 (auto from title) → h2 → h3

### Interactive Tools

- Located in `src/components/apps/` — pure Astro with `<script is:inline>`
- DOM manipulation via `data-*` attributes, NOT `getElementById`
- IDs generated with `crypto.getRandomValues()`, never hardcoded
- No Preact in tools — Preact is exclusively for `src/components/homelab/`

### Security Constraints

- No inline `<script>` tags (breaks CSP nonce-only policy)
- No inline styles — use UnoCSS or scoped `<style>`
- Use `safeJsonLd()` from `@utils/html` for JSON-LD output
- External links: `rel="external noopener noreferrer"` + `target="_blank"`

### Testing

- Playwright + axe-core for E2E and accessibility
- Tests in `tests/*.spec.ts`
- Run `pnpm test:e2e` after changes
- Stop dev server before running tests: `pkill -f "astro dev"`

### Import Order (ESLint enforced)

1. External packages (`astro:content`, `preact`, etc.)
2. Path aliases (`@components/`, `@utils/`, `@layouts/`)
3. Relative imports (`./`, `../`)

### i18n (Internationalization)

- Site is bilingual (EN/ES) — all UI text uses `t()` from `useTranslations()`
- Never hardcode English strings — use translation keys
- Pattern: `const locale = getLangFromUrl(Astro.url)`, `const t = useTranslations(locale)`
- Interpolation: `t("key", { param: value })` with `{param}` in the translation string
- Client-side scripts: inject translations via `data-*` attributes
- Translation files: `src/i18n/translations/{en,es}/{common,tools}.ts`
- Path helpers: `useTranslatedPath(locale)` for localized URLs
- Formatting: `formatDate()`, `formatNumber()`, `pluralize()` from `@i18n/utils`
- See `docs/I18N_GUIDE.md` for complete reference

### Security Files

- `public/.well-known/security.txt` is PGP clearsigned — **always re-sign after editing**:
  ```bash
  gpg --clearsign --default-key 0A993B268654DBBA52B7E8D3FCF653391E2C91FC public/.well-known/security.txt \
    && mv public/.well-known/security.txt.asc public/.well-known/security.txt
  ```
- Verify with `gpg --verify public/.well-known/security.txt`

### Commit Style

- Conventional commits: `feat:`, `fix:`, `docs:`, `refactor:`, `test:`, `chore:`
- Scope optional: `feat(tools): add hash calculator`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jmrplens)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/jmrplens)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
