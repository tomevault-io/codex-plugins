# GitHub Copilot Instructions for MaxeLBerger.github.io

## Overview

Personal portfolio site for Maximilian Haak — fullstack web developer & AI specialist from Bruckmühl/Rosenheim. Static site (HTML + CSS + vanilla JavaScript), deployed to GitHub Pages.

**Live:** [maximilianhaak.de](https://maximilianhaak.de)

## Architecture

Plain static site — **no build step**, no submodules, no frameworks.

```
.
├── index.html              # Homepage (hero slider + sections)
├── impressum.html          # Legal info
├── datenschutz.html        # Privacy policy
├── assets/
│   ├── css/
│   │   └── main.css        # Stylesheet (~2700 lines, design tokens + theming)
│   ├── js/
│   │   └── main.js         # Slider, theme, i18n, animations (~1340 lines)
│   └── img/                # Backgrounds, favicons, icons, profile, projects, screenshots
├── projects/               # Project detail pages
│   ├── main.css            # Project-page-specific styles
│   ├── aicaptain.html
│   ├── coha.html
│   ├── e46-studio.html
│   ├── imkerei-feuerstein.html
│   ├── shookroko.html
│   └── soundoflvke.html
├── docs/                   # architecture / development / deployment guides
├── tools/                  # Local dev tooling (NOT deployed)
│   └── mcp-portfolio-server/   # Local MCP dev tool
└── .github/workflows/
    └── deploy.yml          # Single-job deploy to GitHub Pages
```

## Tech Stack

- **HTML5 / CSS3 / vanilla JavaScript (ES6+)** — no React, no Vue, no bundler
- **GSAP 3.12 + ScrollTrigger** via CDN for animations
- **Inter** (Google Fonts) — loaded after cookie consent (GDPR)
- **i18n**: custom DE/EN dictionary in [assets/js/main.js](../assets/js/main.js) (DE is default)
- **Hosting**: GitHub Pages with custom domain (`CNAME` → `maximilianhaak.de`)

## Core Architectural Concepts

### Homepage hero

The top `#hero` uses the E46 photo as the single full-bleed background. Keep the responsive
`assets/img/backgrounds/hero-e46-640.webp`, `hero-e46-960.webp`, `hero-e46-1440.webp`, and full
`hero-e46.webp` variants wired through the `.hero-bg-squared` `<picture>` sources; the active image is the LCP element,
so keep it eager with `fetchpriority="high"` and do not add competing CSS-background preloads;
do not switch back to the older `herosquared` hero. Keep the dedicated `.hero-section--square` and
`.hero-bg-squared` crop/veil rules. Place the hero text in the right-side grid column (`.hero-section--single`) on
wide screens and use the darker right-side/bottom contrast veil for readability over the E46 photo.
Hero credibility badges belong inside `.hero-text` below the headline as normal `.hero-badges` flex content. Keep them
as standalone white SVG icons with short text labels and without pill/card backgrounds; do not position them as absolute
overlays over the photo. Keep the `Haak.` word in the hero headline as themed `.gradient-text` using the active color
palette, with only a subtle drop-shadow for photo contrast. Animate these `.hero-badge` items as part of the homepage
hero entrance sequence.

### About section

`#about` sits directly after `#projects` and before the tech stack. Keep it as a personal editorial section with an
E46 media frame, short biographical copy, and compact facts. The current media asset is the optimized
`assets/img/profile/uebermich.webp` (portrait, 1000×1334, keep under ~300 KB) in a `.about-image-frame`; preserve the
lower focus so the car/person stay visible. Keep this image large and cleanly cropped in a restrained 5:6 media frame
with subtle border/shadow and no visible caption. The copy column should stay intentionally narrow for comfortable
reading (about 54ch max), rather than stretching across the available grid width. Use
`Über mich` / `About me` as the real section heading, not as an eyebrow; section-note/hint callouts belong only to
`#projects`. If this later becomes video, use
`assets/video/e46-about.mp4` with a poster fallback. All visible copy still goes through `data-i18n` keys in
`assets/js/main.js`; translated image alt text uses `data-i18n-alt`.

### Hero project slider

`#projects` contains exactly ONE `.hero-slides-container` dedicated to own projects: E46 Studio, AI Captain,
Medieval TD, Shookroko, and dog-kennel-online. Above the slider sits a two-option `.project-mode-selector`
segmented pill with
`Kundenprojekte` (left) and `Eigene Projekte` (right). `Eigene Projekte` is the active, enabled option and controls
the single slider. `Kundenprojekte` is intentionally locked (`disabled`, `aria-disabled="true"`, `tabindex="-1"`,
`.is-locked` modifier with a small lock SVG and a visually-hidden `.project-mode-status`) until paid customer work
exists; do NOT wire it up or render customer slides on the homepage. The customer i18n keys (`slide.imkerei.*`,
`slide.coha.*`, `slide.soundoflvke.*`, `slide.danielbrecheis.*`, `slide.kayaseeds.*`, `slide.jkentertainment.*`) are
intentionally retained in [assets/js/main.js](../assets/js/main.js) for future use.

The `ProjectSlider` class in [assets/js/main.js](../assets/js/main.js) handles:

- GSAP-powered transitions (with CSS fallback)
- Touch/swipe + keyboard navigation
- Per-slide theme switching via `data-theme` → `data-project-theme` on `<html>`
- Container height calculation (slides have varying heights)

`ProjectSlider` filters `.project-nav-btn` elements that are `disabled` or have `aria-disabled="true"` out of
`this.navBtns`, so any locked nav option (e.g. the future customer-mode tab) cannot become an active index.

### Theming

Two independent attributes on `<html>`:

| Attribute | Values | Controlled by |
|-----------|--------|---------------|
| `data-color-scheme` | `dark` (default), `light` | Theme toggle button (`#themeToggle`) |
| `data-project-theme` | `maxhaak`, `imkerei`, `coha`, `aicaptain`, `e46`, `medieval`, `dogkennel`, `soundoflvke`, `shookroko` | Slider, scroll observer, and color picker |

The `themeController` IIFE in `assets/js/main.js` is the **single writer** for `data-project-theme`. All callers go
through `setProjectTheme(theme, source)`. The color picker writes are persisted to `localStorage('themeColor')`; slider
writes are not. Picker writes also dispatch the internal theme-change event consumed by `ProjectSlider`, so selecting a
swatch before scrolling to `#projects` activates the matching project slide instead of falling back to the first slide.

`e46` slides reuse the `maxhaak` swatch (no dedicated picker option).

### i18n

All user-visible text uses `data-i18n="key"` attributes. The dictionary lives in `translations.de` and `translations.en` inside [script.js](../assets/js/main.js). On `init()` and on language toggle, `applyTranslations(lang)` overwrites the inline HTML text.

**Critical:** when changing visible text, update **both** the inline HTML default **and** the matching key in both language objects, or the JS will overwrite your HTML change on next load.

Language-neutral technology, product, and tool names in `.skill-tag` / `.tech-badge` labels stay inline as brand
labels. Keep i18n keys for surrounding headings, descriptions, and copy that changes between DE and EN.

Language is persisted in `localStorage('lang')`.

### Performance gates

- **Mouse parallax** in `assets/js/main.js` skips work when `#projects` is off-screen (`parallaxTargetsVisible` flag, fed by an `IntersectionObserver`).
- **Portfolio orb CSS animations** live in `#projects` as `.project-orb` elements and are paused while the section lacks `.in-view`.
- **Hero photo crossfade** (`setInterval`) skips work when `document.hidden`.
- **Google Fonts** are only loaded after the user accepts the cookie banner.
- **Contact location** uses a lightweight external Google Maps link card, not an embedded iframe. Do not reintroduce a
    map iframe on the homepage unless there is a strong product reason and it remains opt-in/lazy.

### Contact form

Two modes detected from the form's `action` attribute:

- `mailto:` → builds a pre-filled email and opens the user's mail client
- HTTP URL → POSTs `FormData` to the endpoint (e.g. Formspree)

Status feedback uses `.is-success` / `.is-error` classes on `.btn-primary` (defined in [style.css](../assets/css/main.css)).

## Deployment

[.github/workflows/deploy.yml](workflows/deploy.yml) — single job:

1. Checkout
2. Copy `index.html`, `*.html`, `CNAME`, `assets/`, `projects/` into `dist/`
3. Upload as Pages artifact
4. Deploy to `github-pages` environment

Push to `main` → live in ~1–2 minutes. There is **no build step** and **no matrix**.

## Coding Guidelines

### General

- Keep it simple — vanilla HTML/CSS/JS only
- Mobile-first responsive design
- Optimize images: WebP preferred, target <300 KB each
- Asset paths must match the exact on-disk filename casing; GitHub Pages is case-sensitive even if local Windows dev is not
- Use semantic HTML and ARIA where appropriate
- No `console.log` in production code
- No inline `style="..."` for state changes — use CSS classes (`.is-success`, `.is-error`, `.active`, etc.)

### CSS

- Use CSS custom properties from `:root` (design tokens)
- Theme overrides go under `[data-project-theme="..."]`
- Light-mode overrides go under `html[data-color-scheme="light"]`
- Mobile-first media queries

### JavaScript

- ES6+ (const/let, arrow functions, template literals, optional chaining)
- Wrap all module-level code in the existing IIFE in `assets/js/main.js`
- Cache DOM lookups; prefer event delegation
- Use `IntersectionObserver` to gate scroll-driven work
- Use `transitionend` / `animationend` instead of `setTimeout` magic numbers

### i18n

When adding or changing visible text:

1. Set `data-i18n="section.key"` and put the German text inline in HTML
2. Add the same key to both `translations.de` and `translations.en` in `assets/js/main.js`

## Files NOT Tracked in Git

See [.gitignore](../.gitignore). Important exclusions:

- `node_modules/`, `dist/`, `build/` (build output)
- `.env*` (secrets)
- `.vscode/`, `.idea/` (IDE)
- Internal planning markdowns (`ANIMATION_PLAN.md`, `LLM_REFACTORING_PLAN.md`, etc.)
- `diff.txt`, `git_log_output.txt` (debug dumps)
- `desktop.ini` (Windows folder metadata)
- `.github/agents/`, `.github/instructions/`, `.github/skills/` (local Copilot config)

## Common Tasks

### Add a new project

1. Add a new `.hero-slide` to `#projects` in [index.html](../index.html) with a new `data-theme`
2. Add a new `.project-nav-btn` to the slider tablist
3. Add the project image to `assets/img/projects/` (optimize to WebP, <300 KB)
4. Add a color theme block to [style.css](../assets/css/main.css) under `[data-project-theme="..."]`
5. Add `slide.<project>.t1/t2/t3/desc/cta1/cta2/badge/tag1/tag2/tag3` keys to both `translations.de` and `translations.en` in [script.js](../assets/js/main.js)
6. Add the project slug to `COLOR_THEMES` in `assets/js/main.js` if it gets a picker swatch
7. Create `projects/<slug>.html` for the detail page
8. Test locally and push

### Update an image

- Always optimize first (WebP, <300 KB target)
- Update the `<img src>` reference in `index.html` and any project detail page
- Delete the old image from `assets/img/projects/`

### Change a translation

- Update both the inline HTML default and both language objects in `assets/js/main.js`

## Local Development

```powershell
python -m http.server 8000
# or
npx serve .
```

Open http://localhost:8000.

## What NOT to Do

- ❌ Don't introduce a build tool (Webpack, Vite, etc.) — site is intentionally buildless
- ❌ Don't add inline `style="..."` for hover/active/state styling
- ❌ Don't commit images >500 KB
- ❌ Don't add new top-level documentation files — extend this file or `README.md` instead
- ❌ Don't use `setTimeout` to wait for CSS transitions — use `transitionend`
- ❌ Don't query the DOM inside animation frames or mousemove handlers — cache references
- ❌ Don't add tracking, analytics, or third-party scripts without updating `datenschutz.html`

---

**Last updated:** May 2026

---
> Source: [MaxeLBerger/MaxeLBerger.github.io](https://github.com/MaxeLBerger/MaxeLBerger.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-12 -->
