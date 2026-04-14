# AI Python Coding Rules

These rules apply to **all AI-generated Python code** in this repository.
AI assistants (Copilot, Cursor, LLM agents, etc.) must follow them at all times.

---

# Python Version

* All code **must target Python 3.14 or newer**.
* Use modern Python features where appropriate.
* Avoid deprecated or legacy syntax.

---

# Import Rules

* **Inline imports are not allowed.**
* All imports must appear **at the top of the module**.
* Imports should be grouped and ordered according to best practices.

Correct example:

```python
from pathlib import Path
import json
import typing as t
```

Incorrect example:

```python
def load_data():
    import json
```

---

# Comment Rules

Comments must explain **intent or reasoning**, not implementation mechanics.

The following are **not allowed**:

* Comments referencing **line numbers**
* Comments describing **what changed on a specific line**
* Comments such as:

```python
# changed line 42
# fixed line 18
# update on line 10
```

Acceptable comments explain **why** something exists.

Example:

```python
# Validate user input before performing database operations
```

---

# Code Quality Standards

All generated code must:

* Follow **PEP 8**
* Be **readable and maintainable**
* Prefer **explicit code over clever code**
* Avoid unnecessary complexity
* Use descriptive variable and function names
* Follow clean architecture principles where applicable

---

# Type Hints

* Use **type hints** for all public functions and methods.
* Prefer **explicit typing** instead of `Any` when possible.
* Ensure the code passes **MyPy type checking**.

Example:

```python
def load_config(path: str) -> dict[str, str]:
    ...
```

---

# Formatting and Linting Requirements

All generated code must be compatible with the following tools:

* **Black** – formatting
* **Ruff** – linting
* **MyPy** – static type checking
* **Flake8**
* **Pylint**

Code should pass these tools **without requiring manual fixes**.

---

# Testing Rules

Tests must follow **pytest best practices**.

Requirements:

* Tests must be **deterministic**
* Tests must be **isolated**
* Avoid unnecessary mocking
* Use clear and descriptive test names

Example:

```python
def test_user_creation_returns_valid_id() -> None:
    ...
```

The following are **not allowed in tests**:

* Comments referencing line numbers
* Inline imports
* Fragile assertions tied to specific implementation lines

---

# General AI Instructions

When generating Python code:

1. Ensure the code is compatible with **Python ≥3.14**.
2. Place **all imports at the module level**.
3. Never include comments referencing **line numbers**.
4. Ensure compatibility with **Black, Ruff, MyPy, Flake8, and Pylint**.
5. Prefer **clean, production-grade Python code**.
6. Follow **Python best practices and PEP 8** at all times.
7. Write code that is **clear, typed, and maintainable**.

---

# Priority of Rules

If an AI tool generates code that violates these rules, the rules in this document **override the generated output**.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mymi14s) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
