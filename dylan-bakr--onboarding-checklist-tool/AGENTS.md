# Copilot Instructions

## Project Overview

This is a React 19 + TypeScript + Vite single-page application for creating and exporting employee onboarding checklists. State is managed with Zustand (persisted to localStorage). PDFs are generated with jsPDF + jspdf-autotable. The app is containerised with Docker and served via Nginx.

Key source locations:

- `src/components/` — React UI components
- `src/store/appStore.ts` — Zustand store (single source of truth)
- `src/data/masterList.ts` — master task database
- `src/utils/pdfExport.ts` — PDF generation logic

## Change Philosophy

**Minimal, targeted changes.** Only touch files that are directly required to fulfil the request. Do not refactor, reorganise, rename, or "improve" surrounding code that isn't broken. Do not add comments, docstrings, or type annotations to code you didn't author. Do not extract helpers or abstractions for one-off operations.

**Comprehensive implementations.** When a change is made, implement it fully — no placeholders, no `// TODO`, no half-finished branches. Edge cases that are reachable from the UI must be handled.

**Thorough investigation first.** Before writing a single line of code, read every file that is relevant to the request. Understand existing patterns (state shape, component conventions, utility helpers) and follow them exactly. If the root cause of a bug is unclear, keep investigating until it is clear — do not guess.

## Development Workflow

All changes must pass the pre-commit hook before being committed. The hook runs `lint-staged`, which executes:

1. `prettier --write` on changed `src/**/*.{ts,tsx,css}` files
2. `eslint --fix` on changed `src/**/*.{ts,tsx}` files

**Before committing, verify:**

```bash
# 1. Confirm the dev server starts without errors
npm run dev
# (open localhost:3000, confirm no console errors, then stop with Ctrl-C)

# 2. Run the full lint suite (mirrors lint-staged but against all files)
npm run lint

# 3. Run type-checking
npx tsc -b --noEmit

# 4. Confirm formatting is clean
npm run format:check
```

All four checks must pass before pushing. If any fail, fix them before committing — do not use `--no-verify`.

## Code Conventions

- **TypeScript**: strict mode. Prefer explicit types for function signatures; let inference handle local variables.
- **React**: functional components with hooks only. Keep components in `src/components/`.
- **State**: all persistent state lives in the Zustand store (`appStore.ts`). Do not use `useState` for data that needs to survive navigation or be shared across components.
- **Styling**: Tailwind CSS v4 utility classes only — no inline `style` props, no separate CSS files (except `index.css` for globals).
- **Imports**: use relative paths within `src/`; no path aliases.
- **Formatting**: Prettier handles all formatting — do not manually adjust whitespace or line breaks.
- **ESLint config**: flat config in `eslint.config.js`. Do not add `eslint-disable` comments without a precise, documented reason.

## Pull Requests

When opening a pull request, always populate the body using the template in `.github/pull_request_template.md`. Fill in every section that applies — remove sections that are genuinely not applicable. Always link the resolving issue by replacing `#0` with the real issue number, or remove the line if there is no related issue.

Attach a screenshot of the relevant change from the dev server when it makes sense (if new frontend changes, screenshot -- if javascript functions change, no screenshot, etc.). Place it in a `## Screenshot` section above the developer notes.

## Documentation & Changelog

Apply the following updates whenever a change warrants them — do not skip them silently:

- **`CHANGELOG.md`** — Add an entry under the appropriate heading (`Added`, `Changed`, `Removed`, `Fixed`) in an `## Unreleased` section at the top (create it if absent). Follow the existing [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) format. Only skip this if the change is purely internal tooling with no user-visible effect.
- **`README.md`** — Update if the change affects features, the tech stack table, setup steps, or usage instructions.
- **`.devcontainer.json`** — Update if the change affects the development environment (Node version, ports, extensions, post-create commands).
- **`.github/copilot-instructions.md`** — Update if the change introduces new project conventions, new key files, or alters the development workflow so that future instructions would otherwise be stale or misleading.

## What NOT to Do

- Do not add new dependencies without a strong justification.
- Do not introduce a test framework — there are currently no tests in this project.
- Do not modify `vite.config.ts`, `tsconfig*.json`, `eslint.config.js`, `nginx.conf`, or `Dockerfile` unless the request is explicitly about those files.
- Do not reformat files wholesale; let Prettier handle that on the changed lines only.
- Do not bypass the pre-commit hook (`--no-verify`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dylan-bakr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
