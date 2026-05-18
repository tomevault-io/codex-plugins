# Azhar Ali Portfolio — AI Agent Instructions

## Project overview
- Static, multi-page portfolio site (HTML + CSS + JS) with production build outputs in dist/.
- Source assets live under assets/ (CSS/JS/images) and js/ (feature scripts).
- Security features (CSRF, rate limiting, headers) are documented in repo guides; see [CSRF-PROTECTION-GUIDE.md](CSRF-PROTECTION-GUIDE.md), [RATE-LIMITING-GUIDE.md](RATE-LIMITING-GUIDE.md), and [SECURITY-IMPLEMENTATION-COMPLETE.md](SECURITY-IMPLEMENTATION-COMPLETE.md).

## Build & dev workflows (from package.json)
- Build all: npm run build (cleans dist/ then bundles JS and minifies CSS).
- Watch: npm run watch (passes --watch into build scripts).
- Local serve: npm run serve (http-server on :8080).
- JS bundling: [build-scripts/build-js.js](build-scripts/build-js.js) (esbuild, iife, sourcemaps) → dist/js/*.min.js.
- CSS minify: [build-scripts/build-css.js](build-scripts/build-css.js) → dist/css/main.min.css.

## Source layout & patterns
- HTML entry points: index.html, projects.html, contact.html, skills.html, security-lab.html.
- CSS is modularized in assets/styles/ with base/, components/, layout/ and variables in assets/styles/base/variables.css; build script concatenates these into one bundle.
- JS is split between assets/scripts/ (site-wide behaviors) and js/ (feature modules like security, particles, forms). Production bundles are defined in [build-scripts/build-js.js](build-scripts/build-js.js).
- Production HTML should reference dist/ bundles (see build script “Next steps” output); dev HTML may reference source files.

## Key feature entry points
- Core site behavior: assets/scripts/main.js and assets/scripts/performance.js.
- Security/CSRF form flow: js/form-validation.js, js/input-sanitizer.js, js/message-encryption.js, and js/production-config.js.
- Security lab demo: js/security-simulator.js and js/skill-demos.js.

## Conventions to follow
- Keep new styles in the existing base/components/layout structure under assets/styles/; avoid editing dist/ directly.
- Update bundle entry points in [build-scripts/build-js.js](build-scripts/build-js.js) if new JS should be included in production bundles.
- When changing asset locations, update HTML links and rebuild to regenerate dist/.

## References
- Deployment & production guidance: [PRODUCTION-QUICK-START.md](PRODUCTION-QUICK-START.md), [PRODUCTION-DEPLOYMENT-GUIDE.md](PRODUCTION-DEPLOYMENT-GUIDE.md).
- Security test page: [csrf-test-suite.html](csrf-test-suite.html).

---
> Source: [FaheemAkbar12/Portfolio](https://github.com/FaheemAkbar12/Portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-18 -->
