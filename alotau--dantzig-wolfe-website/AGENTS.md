# dantzig-wolfe-website Development Guidelines

Auto-generated from all feature plans. Last updated: 2026-03-07

## Active Technologies
- TypeScript 5 / Astro 5 / Svelte 5; Node 20 + No new dependencies — static HTML only (003-kofi-donate-button)
- Node.js 22 (pinned in `ci.yml`); TypeScript via Vitest for tests + Node.js built-ins only (`fetch`, `crypto`, `fs/promises`) — no new npm packages (004-solver-wheel-ci)
- Filesystem — `public/dwsolver-*.whl` (downloaded at build time, gitignored) (004-solver-wheel-ci)

- **Frontend**: Astro 5.x (SSG, island architecture), Svelte 5.x (interactive islands), Tailwind CSS 4.x
- **Solver runtime**: Pyodide 0.29.x (Python 3.12 on WASM in-browser Web Worker)
- **Solver package**: `dantzig-wolfe-python` (https://github.com/alotau/dantzig-wolfe-python) via micropip
- **Math notation**: KaTeX (server-side via rehype-katex in MDX; client-side in solver output)
- **Charts**: Chart.js 4.x
- **Testing**: Vitest 3.x (unit), Cucumber.js 11.x + Playwright 1.50.x (BDD acceptance)
- **Language**: TypeScript 5.x (site); Python 3.12 runs only inside Pyodide WASM

## Project Structure

```text
features/          Gherkin feature files (authoritative specifications — edit first)
src/
  pages/           Astro page routes
  components/      Astro + Svelte components (solver/, layout/, content/, examples/)
  content/         MDX educational content collections (history, lessons, examples, glossary)
  workers/         solver.worker.ts — Pyodide Web Worker
  lib/             solver/worker-client.ts, solver/problem-schema.ts, sharing/url-codec.ts
  styles/          global.css — Tailwind + CSS custom properties
tests/
  unit/            Vitest tests for lib/ utilities
  acceptance/      Cucumber.js step definitions + Playwright browser support
public/
  pyodide-lock.json  Pinned dependency lockfile
  examples/          Pre-built problem JSON files
specs/
  002-initial-feature-files/  plan.md, research.md, data-model.md, quickstart.md, contracts/
```

## Commands

```bash
npm run dev              # Astro dev server at localhost:4321
npm run build            # Production static build → dist/
npm run preview          # Serve dist/ locally
npm run test:unit        # Vitest unit tests
npm run test:acceptance  # Build + Cucumber.js + Playwright acceptance tests
npm run test             # All CI gates: audit → lint → unit → acceptance
npx playwright install --with-deps chromium  # First-time Playwright setup
```

## Code Style

- TypeScript strict mode; no `any`; Zod for all runtime validation at trust boundaries
- Svelte 5 runes (`$state`, `$derived`, `$effect`) — no legacy Options API
- Tailwind utility classes; design tokens as CSS custom properties in `src/styles/global.css`
- All math rendering via KaTeX; never raw `innerHTML` from user input
- Web Worker is the ONLY place Pyodide/Python runs; no solver logic on the main thread

## Key Constraints (Constitution)

- **Never push directly to `main`** — all changes via PR on a numbered branch
- **Always merge `main` into working branch before pushing**
- **Gherkin first** — feature files in `features/` precede implementation
- **No server-side computation** — the solver runs only in the browser via Pyodide

## Recent Changes
- 004-solver-wheel-ci: Added Node.js 22 (pinned in `ci.yml`); TypeScript via Vitest for tests + Node.js built-ins only (`fetch`, `crypto`, `fs/promises`) — no new npm packages
- 003-kofi-donate-button: Added TypeScript 5 / Astro 5 / Svelte 5; Node 20 + No new dependencies — static HTML only


<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alotau) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
