type: auto_attached
name: Python Project Standards
pattern: "**/*.py"

rules:
  - description: "If no venv is detected (.venv or venv folder), create one"
    command: "python -m venv .venv"

  - description: "For each Python task, check that it aligns with a task in @roadmap.txt"

  - description: "Run pytest after code changes and report the test results"
    command: "pytest"

  - description: "Run flake8 and include the linting output"
    command: "flake8"

  - description: "Only mark a task complete if all tests pass and there are zero flake8 errors"

  - description: "Log actions in @history.txt with a timestamp (e.g., '2025-05-04: Added login function')"

  - description: "Use a first-principles approach to problem solving. Avoid unnecessary abstractions and over-engineering"

  - description: "Check for a .env file. If it exists, assume it contains API keys. Do not hardcode any credentials or prompts"

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vyente-ruffin)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/vyente-ruffin)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
