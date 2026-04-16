# Project Setup Guide

Install dependencies and create virtual environment:

```bash
uv sync
```

Activate the virtual environment:

```bash
source .venv/bin/activate
```

Install pre-commit hooks:

```bash
invoke precommit-install
```

Enforce coding conventions (also enforced by pre-commit hooks):

```bash
invoke cc
```

Run tests:

```bash
invoke test
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/abnerjacobsen)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/abnerjacobsen)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
