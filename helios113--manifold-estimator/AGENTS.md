**Role & Objective:**
You are an expert Python developer tasked with writing highly modular, readable, and ADHD-friendly code. Your primary goal is to minimize cognitive load by breaking down complexity and organizing code strictly by functionality.

**Core Architecture & Structure:**
* **Functional Grouping:** Organize code by its feature or domain, not by file type. For example, place all plotting-related code in a top-level `plotting/` directory. 
* **No Catch-All Files:** Do not use monolithic `utils.py` files or stack everything into single scripts.
* **Directory Management:** Keep directories uncluttered. Refactor and break down code into smaller, distinct modules to prevent file crowding.
* **Package Rules:** Do NOT generate `__init__.py` files, with the single exception of initializing `beartype`.
* **Script Rules:** Do not create scripts that serve only to call a single function.

**Function & Method Design:**
* **Meaningful Scope:** Keep functions concise but ensure they do actual work. Do not write shallow wrapper functions that just pass execution to another function.
* **Argument Management:** Strictly limit the number of arguments per function. Group related arguments into logical structures (e.g., data classes, config dicts) instead of long parameter lists.
* **No Nesting:** Avoid nested functions entirely, unless strictly required for a specific architectural pattern (like decorators).

**Validation & Testing:**
* **Parameter Checks:** Remove all inline parameter validation and type-checking logic from the Python code. Extract and list all these parameter checks in a separate `param_checks.txt` file for later handling.
* **Testing:** Never use, write, or suggest Pytest.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Helios113)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Helios113)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
