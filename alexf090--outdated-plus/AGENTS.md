# Copilot Instructions for outdated-plus

This guide enables AI coding agents to work productively in the outdated-plus codebase. It summarizes architecture, workflows, and project-specific conventions.

## Architecture Overview
- **CLI Tool**: Extends `npm outdated` by adding publication dates and age info for npm packages.
- **Modes**:
  - **Standard**: Uses `npm outdated` and enriches results with data from the npm Registry API.
  - **Full Check (`--check-all`)**: Directly queries all packages via the npm Registry API, bypassing `npm outdated`.
- **Zero Runtime Dependencies**: Only Node.js built-ins are used at runtime. All external packages are dev-only.
- **Key Directories**:
  - `src/`: Main source code. CLI entry is `src/index.ts`. Core logic in `src/lib/` (e.g., `output.ts`, `processing.ts`, `concurrency.ts`).
  - `tests/`: Vitest-based tests for all major modules.
  - `scripts/`: CI and utility scripts.

## Developer Workflows
- **Build**: `npm run build` (TypeScript, outputs to `dist/`)
- **Test**: `npm test` or `npx vitest`
- **Start (dev)**: `npm run start` (runs CLI locally)
- **Debug**: Build locally for source maps; see `dist/` after `npm run build`.
- **CI**: See `scripts/ci-local-all.sh` for local CI simulation.

## Project-Specific Conventions
- **No runtime dependencies**: All logic uses Node.js built-ins (`fs`, `path`, `child_process`, native `fetch`).
- **Skip logic**: Packages can be skipped via CLI flag or `.outdated-plus-skip` file. Auto-cleanup removes obsolete skip entries.
- **Output**: Results are tabular, with optional Markdown formatting (`--format md`).
- **Sorting**: Multiple sort fields; see `--sort-by` option in README.
- **Concurrency**: Controlled via `--concurrency`, clamped 1-100.
- **Error Handling**: Exit code `1` for errors, `0` for success.

## Integration Points
- **npm CLI**: Used for `npm outdated` in standard mode.
- **npm Registry API**: All metadata fetched live, no caching.
- **Filesystem**: Reads `package.json`, `package-lock.json`, `.outdated-plus-skip`.

## Patterns & Examples
- **Main entry**: `src/index.ts` parses CLI args and orchestrates flows.
- **Core logic**: See `src/lib/processing.ts` for package analysis, `src/lib/output.ts` for formatting.
- **Testing**: Each module in `src/lib/` has a corresponding test in `tests/`.

## License & Restrictions
- **Proprietary**: Modification, forking, and integration into other projects is prohibited.

---
For unclear conventions or missing details, ask the user for clarification or examples from the codebase.

---
> Source: [AlexF090/outdated-plus](https://github.com/AlexF090/outdated-plus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-31 -->
