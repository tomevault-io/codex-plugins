# Python Package Management with uv

Use uv exclusively for Python package management. Never use pip, pip-tools, poetry, or conda directly.

## Commands

- Install: `uv add <package>`
- Remove: `uv remove <package>`
- Sync: `uv sync`
- Run scripts/tools: `uv run <script>.py` or `uv run pytest`
- Python REPL: `uv run python`

## Monorepo Usage

**Always run commands from the repo root** (where `uv.lock` lives). Use path arguments to target services:

- Tests: `uv run pytest services/{name}/tests/`
- Linting: `uv run ruff check services/{name}/`

---
> Source: [NVIDIA-NeMo/nemo-platform](https://github.com/NVIDIA-NeMo/nemo-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-22 -->
