# Backend

## Python

Always use `uv` for Python. Never edit `pyproject.toml` directly. Never use pip.

## Linting and formatting

After every bigger code change, run:

```
make check-and-format-py
```

This runs pyrefly (type checker) then black (formatter). Fix any errors before proceeding.

By bigger code we mean that you should not actually run this check every time you edit something, but you should always run it before you try to present the code to the user.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lambdaofgod)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/lambdaofgod)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
