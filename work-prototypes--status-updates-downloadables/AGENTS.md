# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Status updates downloadables — a Python 3.13 project managed with [uv](https://docs.astral.sh/uv/).

## Commands

- **Install deps:** `uv sync`
- **Lint:** `uv run ruff check .`
- **Lint fix:** `uv run ruff check --fix .`
- **Type check:** `uv run pyright`

## Code Style

- Ruff enforces pycodestyle (E), pyflakes (F), and isort (I) with 88-char line length.
- Pyright strict mode is enabled — all code must pass strict type checking.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/work-prototypes)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/work-prototypes)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
