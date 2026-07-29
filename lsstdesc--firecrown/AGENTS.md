# Firecrown Project - GitHub Copilot Instructions

This repository uses a modular instruction system. All detailed coding standards, testing protocols, and formatting rules are stored in the [.github/instructions/](./instructions/) directory.

GitHub Copilot is configured to automatically load all `*.instructions.md` files in that directory.

## Summary of Standards
- **Python**: PEP8 + Black, Sphinx-style docstrings, mandatory type annotations.
- **Testing**: Comprehensive coverage, functional over class-based tests, `make test`.
- **Commits**: 40-character subject line, imperative mood, 72-character body.

For a full overview of the AI agent configuration, see [AGENTS.md](../AGENTS.md).

---
> Source: [LSSTDESC/firecrown](https://github.com/LSSTDESC/firecrown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-24 -->
