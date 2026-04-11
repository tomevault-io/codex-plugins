# GitHub Copilot Instructions

Follow these repository instructions when working in this project.

## General guidance

- Keep changes minimal and aligned with the existing Sphinx-based GitHub Pages site.
- Write all new or updated repository instructions, code comments, and documentation edits in English.
- Do not introduce environment-specific absolute paths, machine-specific settings, or local-only workflows.
- Prefer updating existing pages and configuration instead of adding parallel structures unless there is a clear need.
- Do not commit generated output in `build/` unless the task explicitly requires publishing built artifacts.

## Project context

- Documentation sources live under `source/` and are built with Sphinx.
- Markdown support is provided by MyST, so Markdown content should remain compatible with MyST and Sphinx.
- Project dependencies are managed with `uv` and defined in `pyproject.toml`.
- GitHub Actions workflows under `.github/workflows/` are used for CI and publishing-related automation.

## Validation

- For dependency changes, prefer `uv sync` to align the virtual environment with `pyproject.toml` and `uv.lock`.
- For documentation or configuration changes, prefer validating with `make html` when feasible.
- Clearly distinguish between checks you ran and checks you did not run.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yuokada)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/yuokada)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
