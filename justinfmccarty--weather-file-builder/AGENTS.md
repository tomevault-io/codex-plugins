
---
description: Core project environment and conventions for weatherfilebuilder
alwaysApply: true
---

# weatherfilebuilder Project Environment

## Python Interpreter

Always use the `weatherfilebuilder` conda environment as the Python interpreter.

```bash
# Activate before running anything
conda activate weatherfilebuilder

# Or invoke directly
~/miniconda3/envs/weatherfilebuilder/bin/python
```

When suggesting terminal commands, prefix with `conda run -n weatherfilebuilder` or assume the env is active.

## Project Layout

- `weatherfilebuilder/` — main package source
- `weatherfilebuilder/tools/` — core pipeline modules (configuration, workflow, io, etc.)
- `tests/` — pytest test suite
- `gui/` — GUI front-end (see `gui/README.md`)
- `examples/` — usage notebooks/scripts

## Running Tests

```bash
conda run -n weatherfilebuilder pytest tests/
```

## Package Management

The project uses `hatchling` as the build backend (`pyproject.toml`). Do not add a `setup.py` or `setup.cfg`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/justinfmccarty)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/justinfmccarty)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
