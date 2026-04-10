
# Copilot Instructions for svelte_docinfo_sketch

## Project Overview
- **Purpose:** Experimental tool for extracting metadata from Svelte 5 components, inspired by Sveld but adapted for Svelte 5 and SvelteKit. Not published to npm; intended for experimentation and public domain use.
- **Core logic:** Metadata extraction is performed by parsing Svelte component ASTs using `zimmerframe` and Svelte's `parse` (with `{modern: true}`), without TypeScript inference.

## Architecture & Key Files
- **Metadata Extraction:**
  - [src/lib/metadata.ts](src/lib/metadata.ts): Central parser for extracting props, exports, generics, and JSDoc comments from Svelte components. Extend this file for new metadata features.
  - [src/lib/documentor.ts](src/lib/documentor.ts): (If present) May provide higher-level APIs or utilities for documentation generation.
- **Sample-driven Testing:**
  - [tests/samples/](tests/samples/): Each subfolder contains a Svelte component (`test.svelte`), its parsed AST (`ast.json`), and expected metadata output (`expected.json`). Add new samples here to validate extraction logic.
- **Routes & Config:**
  - [src/routes/package.ts](src/routes/package.ts): Exposes `package.json` as a typed module for routes.
  - [src/routes/package.gen.ts](src/routes/package.gen.ts): Re-exports Gro's package generator for typed package info.
  - [svelte.config.js](svelte.config.js): SvelteKit config, static adapter, root-absolute paths, and aliases (`$routes`, `$tests`).
  - [vite.config.ts](vite.config.ts): Vite config for local dev.
  - [tsconfig.json](tsconfig.json): TypeScript config (no type inference for Svelte props/exports).

## Developer Workflows
- **Build, Test, Lint:**
  - Use Gro for all major workflows:
    - `npm run dev` — Local development
    - `npm run build` — Build project
    - `npm run check` — Type and lint checks
    - `npm run test` — Run tests (validates against [tests/samples/](tests/samples/))
  - GitHub Actions ([.github/workflows/check.yml](.github/workflows/check.yml)) runs `gro check --workspace` and `gro build` on push/PR.
- **Formatting & Linting:**
  - Prettier config is in `package.json` (uses `prettier-plugin-svelte`).
  - ESLint config is in [eslint.config.js](eslint.config.js).

## Project-Specific Patterns & Conventions
- **Metadata Extraction:**
  - Only AST analysis is used; no TypeScript type inference.
  - JSDoc comments are attached to props/exports.
  - Generics are extracted from `<script>` attributes.
  - Extend [src/lib/metadata.ts](src/lib/metadata.ts) for new extraction logic.
- **Testing:**
  - Add new Svelte component samples to [tests/samples/](tests/samples/) with matching `ast.json` and `expected.json` for regression coverage.
- **Aliases:**
  - `$routes` → `src/routes`, `$tests` → `src/tests` (see [svelte.config.js](svelte.config.js)).

## External Dependencies
- `zimmerframe` for AST walking
- Svelte's compiler for parsing
- Gro for build/test/check workflows

---

_If any section is unclear, incomplete, or missing important project-specific details, please provide feedback to improve these instructions._

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/medyll)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/medyll)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
