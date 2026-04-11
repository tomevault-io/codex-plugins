# GitHub Copilot Instructions

## Global Behavior

1. **ALWAYS ASK BEFORE WRITING CODE**
   - Never generate or modify code without first asking for explicit approval.
   - Example: “I can write a function to do X using Y; would you like me to generate the code now?”

2. **Use Poetry for Dependencies and Environments**
   - Assume this project uses **Poetry** for both dependency management and virtual environments.
   - When suggesting how to install dependencies, use:
     - `poetry add <package>` for runtime deps
     - `poetry add --group dev <package>` for dev/test tooling
   - When suggesting how to run commands or scripts, prefer:
     - `poetry run <command>`
   - Avoid suggesting direct `pip install` or manual `venv` management unless explicitly requested.

3. **Present Options as A, B, C, … and ALWAYS Include a Recommendation**
   - When providing multiple options/approaches/strategies, always format them as clearly labeled choices:
     - **A)** …
     - **B)** …
     - **C)** …
   - After listing the options, **always include a recommendation** such as:
     - “**Recommendation:** Option B is best because…”
   - Keep each option concise and focused for easy comparison.

## Style & Interaction

- Be explicit about assumptions (file paths, module names, environment setup).
- When proposing code changes or new files, describe the idea first, then ask for approval before generating code.
- When unsure between multiple approaches, present them as choices (A, B, C) and wait for a selection before proceeding.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kevinflint-cs2)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/kevinflint-cs2)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
