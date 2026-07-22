# Copilot Instructions for Sublimer Log

## Python Guidelines

**Version**: Python 3.8+ (Sublime Text embedded Python)

**Avoid**:
- Union operator `|` for type hints (use `Union` or `Optional` from typing module)

**Use**:
- f-strings for string formatting: `f"Hello, {name}!"`
- Type hints for better code clarity:
  - `Union[str, int]` instead of `str | int`
  - `Optional[str]` instead of `str | None`
- `pathlib` for file operations
- Modern async/await syntax
- Walrus operator `:=`
- `sublime` and `sublime_plugin` modules

## File Naming

**Markdown files**: Always lowercase with hyphens
- ✅ `readme.md`, `changelog.md`, `api-reference.md`
- ❌ `README.md`, `API_Reference.md`

---
> Source: [neverbot/SublimerLog](https://github.com/neverbot/SublimerLog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-22 -->
