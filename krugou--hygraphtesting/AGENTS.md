# GitHub Copilot Instructions

These instructions are used by Copilot and agents when assisting with development in this repository.

always update copilot instructions when making changes to the project structure, coding guidelines, or workflow. This ensures that Copilot can provide accurate and relevant suggestions based on the latest project context.

always also add documentation to readme.md when making changes to the project structure, coding guidelines, or workflow. This ensures that all contributors have a clear understanding of how the project is organized and how to work with it effectively.

## Project Overview

This is a small Node.js project that interacts with Hygraph GraphQL API. It contains scripts for asset management, client operations, mutations, and queries. Dependencies are managed via npm.

## Coding Guidelines

- Follow standard Typescript practices.
- Keep code small and modular; scripts live in the `scripts/` folder.
- Use `npm run lint` to check style and `npm test` (if available) for tests.

## Dependency Management

- Use `npm-check` or `npm outdated` to identify outdated packages.
- Regular dependency updates are encouraged; Dependabot may be configured separately.

## Workflow Notes

- There is no CI configuration in the repo. When adding features, ensure they run locally.
- Branching follows GitHub Flow: create feature branches off `main`.
- Frontend styling uses Tailwind CSS v4 with the `@tailwindcss/vite` plugin.
- Do not reintroduce `tailwind.config.js` or `postcss.config.js` unless explicitly needed.
- Keep `frontend/src/index.css` as the Tailwind entrypoint (`@import 'tailwindcss'`).

## Copilot Usage

- When generating code, prefer small, readable functions.
- Avoid adding unnecessary dependencies.
- If unsure about API usage, consult Hygraph documentation.

Never ask questions just start working and if errors are encountered, try to fix them based on the error messages and project context.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Krugou)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Krugou)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
