# soilgraph — Copilot Instructions

- These rules apply to all Copilot interactions in this repository.
- Don't re-read files you've already read in this session unless I ask you to. Minimize calls to tools and work with what you already have in context.

## Development Workflow

- **Never commit directly to `main`.** Always work in a branch created from an issue.
- Name branches `{type}/{issue-number}-short-description` (types: `feature/`, `fix/`, `docs/`, `test/`, `chore/`).
- Use Conventional Commits for all commit messages: `feat:`, `fix:`, `docs:`, `test:`, `chore:`, `refactor:`.
- Include the issue number in commit messages when applicable: `feat: add gravel support (#15)`.

## Mandatory Checklist (every code change)

Before considering any implementation complete, verify:

1. **Tests** — add or update `testthat` tests for every behavior change. Test normal cases, edge cases, and error paths.
2. **NEWS.md** — add a bullet point under the current development version heading for every user-facing change.
3. **Roxygen documentation** — update `@param`, `@return`, `@examples` when any function signature, behavior, or return value changes.
4. **Examples** — include executable examples in roxygen docs for new exported functions.
5. **`devtools::document()`** — run this whenever roxygen comments change to keep `NAMESPACE` and `man/` in sync.
6. **Vignettes** — update relevant vignettes when adding new features or changing existing behavior.

## Version Bumping

- The version in `DESCRIPTION` follows semantic versioning: `MAJOR.MINOR.PATCH`.
- During development, the version has a `.9000` suffix (e.g., `0.1.0.9000`).
- **PATCH** bump: bug fixes, documentation improvements, internal refactors.
- **MINOR** bump: new features, new exported functions, backward-compatible additions.
- **MAJOR** bump: breaking API changes (renamed/removed exports, changed return types).
- Bump the version in `DESCRIPTION` at release time, not on every commit.

## Code Standards

- Prefer base R or existing dependencies. Do not add new dependencies without clear justification.
- Use `pkg::fun()` when calling imported package functions where it aids clarity.
- Use `TRUE` and `FALSE`, never `T` and `F`.
- Maximum line length: 120 characters.
- Both `snake_case` and `dotted.case` names are accepted (existing convention).
- Favor explicit argument validation, predictable return values, and informative error messages.
- Avoid partial argument matching, `pkg:::fun`, and hidden global state.
- Keep examples lightweight, deterministic, and CRAN-safe (no internet, no interactive prompts).

## Testing Requirements

- Use `testthat` (edition 3) for all tests.
- Tests go in `tests/testthat/`. Advanced/slow tests go in `tests/testthat-advanced/`.
- Keep tests deterministic and isolated.
- Use `skip_on_cran()` only for genuine platform or environment constraints.

## Pre-Merge Requirements

Before any code is merged to `main`:

- `R CMD check` must pass cleanly (no errors, no warnings). Run: `make check`
- `lintr::lint_package()` must pass. Run: `make lint`
- All `testthat` tests must pass. Run: `make test`
- Roxygen docs must be regenerated. Run: `make document`

## Makefile Reference

Use these targets for common operations:

- `make test` — run testthat tests
- `make check` — full R CMD check
- `make document` — regenerate roxygen2 docs and NAMESPACE
- `make lint` — run lintr on package code
- `make style` — auto-format R files with styler
- `make site` — build pkgdown documentation site

## Package Structure

- `R/` — package source code
- `man/` — generated roxygen2 documentation (do not edit manually)
- `tests/testthat/` — standard test suite
- `tests/testthat-advanced/` — extended tests (not run on CRAN)
- `vignettes/` — long-form documentation
- `inst/extdata/` — example data files
- `dev/` — developer setup scripts (not part of the package)

---
> Source: [nriveras/soilgraph](https://github.com/nriveras/soilgraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-29 -->
