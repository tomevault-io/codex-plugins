# Copilot Instructions

## Git Commit Messages
- Prefer short, concise commit messages (one or two lines maximum)
- Combine git add and commit into a single command (e.g., `git add . && git commit -m "message"`)

## Python Environment
- Always use the `.venv` virtual environment for Python commands and dependencies
- Use `.venv/bin/python` when running Python scripts
- Use `.venv/bin/pip` when installing packages
- Use `.venv/bin/mypy` when running mypy type checks

## Testing and Quality Checks
- Always run pytest and mypy before committing changes
- Use `./run_test.sh` for running pytest and mypy
- Both checks must pass before committing code

## Documentation
- Update readme file when adding a significant new feature
- Document complex architectural design decisions in the docs/ directory
- Avoid over-documenting trivial changes
- Keep docstring comments concise and relevant, lean on one-liners

---
> Source: [antondaubert/dreame-mower](https://github.com/antondaubert/dreame-mower) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-05 -->
