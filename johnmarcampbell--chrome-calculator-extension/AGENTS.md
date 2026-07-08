# Copilot Instructions

## Project Overview
This repository contains a Chrome calculator extension with expression evaluation, graphing, and history features.

## Tech Stack
- JavaScript (CommonJS)
- Jest with jsdom for tests
- Browser extension APIs (`chrome.storage.local`)

## Coding Guidelines
- Keep changes minimal and focused on the user request.
- Preserve existing module boundaries and exported APIs unless a change explicitly requires API updates.
- Prefer clear, small helper functions over large inline logic blocks.
- Maintain current naming and style conventions in each file.

## Testing Expectations
- Utilize test-driven development (TDD) for new features and bug fixes. Write a single failing test that captures a small piece of functionality, then implement the minimum code to pass that test before moving on to the next test.
- For bug fixes, write a test that reproduces the bug before implementing the fix.
- Ensure all existing tests pass after your changes. If you encounter test failures, investigate and resolve them before finalizing your changes.
- Run the full test suite before finishing:
  - `make test`
- For substantial logic changes, also run coverage:
  - `make test-coverage`

## Evaluator-Specific Notes
- Normalize input safely before parsing.
- Keep error messages stable when possible to avoid breaking tests and UX.
- Handle parser and runtime failures predictably.

## History and UI Behavior
- Preserve raw user expression text for history display.
- Avoid changing UI behavior unless requested.

## Scope Discipline
- Do not introduce new dependencies unless clearly necessary.
- Do not refactor unrelated files.
- If requirements are ambiguous, ask for clarification before broad changes.

---
> Source: [johnmarcampbell/chrome_calculator_extension](https://github.com/johnmarcampbell/chrome_calculator_extension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-07 -->
