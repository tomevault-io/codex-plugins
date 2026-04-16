# Copilot Instructions for sybilsedge

This is a personal portfolio and dashboard site for Sybil Melton — writer, cloud architect, maker, and cook.

## Stack

- **Framework:** Astro v6 (`output: 'server'`) with the `@astrojs/cloudflare` adapter — deployed as a Cloudflare Worker
- **UI layer:** React 19 for interactive islands; `.astro` components everywhere else
- **Styling:** Tailwind CSS v4 via `@tailwindcss/vite` — there is no `tailwind.config.js`; all config lives in CSS or `vite.plugins`
- **TypeScript** throughout; strict mode is expected
- **Icons in React:** `lucide-react` (e.g., `AlertTriangle`, `CheckCircle2`)
- **Icons in `.astro`:** inline SVG strings rendered with Astro's `set:html` directive — this is intentional and not an XSS risk because all SVGs are locally authored

## Conventions

### Components
- Static/layout components → `.astro` files in `src/components/`
- Interactive components → `.tsx` files in `src/components/`, mounted with a `client:*` directive (prefer `client:visible` for below-the-fold islands)
- All pages use the `Layout.astro` wrapper and should pass `title`, `description`, and optionally `image` props for SEO

### Icons in `.astro` files
Icons are stored as inline SVG strings in a `Record<string, string>` map and rendered via `<span set:html={icons[key]} />`. New icons should follow this pattern — do not introduce `<img>` tags or external icon component libraries in `.astro` files.

### Styling
- Use Tailwind utility classes; avoid inline `style` attributes unless Tailwind cannot express the value
- Typography scale: `font-orbitron` for headings/wordmarks, `font-tech` for labels and metadata, standard sans for body prose
- Neon accent color: `#39ff14` (neon green) and `cyan-300` family — keep additions consistent with the dark terminal aesthetic

### Content Collections
Content schemas are defined with Zod in `src/content.config.ts`. There are four collections: `writing`, `projects`, `recipes`, `posts`. When adding a new collection or field:
- Follow the existing `defineCollection` + `z.object()` pattern
- Use `.optional()` for fields that may not be present on every entry
- Use `z.enum([...])` for fields with a fixed set of values

### Cloudflare Workers constraints
The SSR adapter targets `webworker` (`vite.ssr.target: 'webworker'`). Flag any code that uses Node-only APIs (`fs`, `path`, `crypto` from Node, `process.env` without a Cloudflare-compatible shim, etc.) — these will break at runtime on the edge.

## Security

### `set:html` scope
`set:html` is only safe for locally authored, static strings (e.g., the inline SVG icon maps). Never pass user-supplied data, API responses, URL parameters, or any dynamically constructed string to `set:html` — it bypasses Astro's HTML escaping and will create an XSS vector.

### `dangerouslySetInnerHTML` in React
Never use `dangerouslySetInnerHTML` with content that comes from user input, API responses, or URL/search parameters. If rich HTML output is genuinely needed, sanitize it first with a library like `dompurify` before passing it in.

### Secrets and environment variables
- **Runtime secrets** (API keys, tokens, credentials) must be stored as Cloudflare secrets/bindings and accessed via `Astro.locals.runtime.env.SECRET_NAME` inside SSR pages and endpoints.
- **Build-time public values** use `import.meta.env.PUBLIC_*` (these are inlined into the client bundle — never put secrets here).
- `process.env` is not available in the `webworker` SSR target and must not be used.
- Never hardcode secrets or tokens in source files.

### Security headers
Cloudflare Workers serve static headers from `public/_headers`. This file is the correct place to set:
- `Content-Security-Policy`
- `X-Frame-Options: DENY`
- `X-Content-Type-Options: nosniff`
- `Referrer-Policy: strict-origin-when-cross-origin`
- `Permissions-Policy`

Do not set these headers in component or page code.

### Updating the CSP
Security response headers are configured in `public/_headers` under the `/*` route.

If you add any new external resource (font, script, API, image CDN), you **must** update
the `Content-Security-Policy` value or the resource will be silently blocked in production.

| Resource type | Directive to update |
|---------------|-------------------|
| External font | `style-src` + `font-src` |
| External script | `script-src` |
| API / fetch call | `connect-src` |
| External image | `img-src` |

After updating, re-run Lighthouse Best Practices on the preview URL to confirm the score
stays at 100 before merging to main.

### MDX is code, not content
All four content collections accept `.mdx` files, which execute JavaScript at build time. Treat new MDX-imported components as code — they require the same review as any `.tsx` file. If user-submitted content is ever supported in the future, it must never be rendered as MDX without a dedicated sanitization step.

### Zod URL fields
All new URL fields in content schemas must use `z.string().url()` (not bare `z.string()`). This blocks malformed values and `javascript:` URIs. See `githubUrl` in `src/content.config.ts` as the reference pattern.

## What to skip reviewing
- `README.md` — this is the unmodified Astro starter template and is not kept up to date
- `package-lock.json` diffs — no review needed unless a specific dependency concern is raised

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sybilsedge) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
