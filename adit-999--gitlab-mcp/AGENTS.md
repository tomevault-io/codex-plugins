# GitLab MCP Project - GitHub Copilot Instructions

## Core Principles
- Target Python 3.13 with all code.
- Adhere strictly to **PEP8** for styling and **PEP585** for type hinting with built-in generics.
- Use **double quotes** for strings and docstrings as per project ruff configuration.
- Follow **88 character line length limit** as defined in project settings.
- Prioritize **explicitness over implicitness** in all code.
- Follow the **Principle of Least Surprise** in API and interface design.
- **Avoid inline comments** - prefer descriptive variable names and function names instead.

## Code Structure
- Favor **small, focused `@dataclass`es** over complex or mutable classes.
- Limit function arguments to **maximum of 7 parameters** as configured in pylint settings.
- Enforce a **maximum of 30 statements per function** per pylint configuration.
- Use **typestate and fluent interfaces** where appropriate for ergonomic usage.
- Organize imports according to the isort configuration with order: future, standard-library, third-party, first-party, local-folder.
- Combine imports with the same namespace (e.g., `from package import Class1, Class2`).

## Type System
- Require **specific type annotations** throughout, using Python 3.13 features.
- Use **pathlib** for all file path operations as enforced by the PTH linting rule.
- Use `Protocol` classes for **structural typing** when interfaces matter more than inheritance.
- Prefer **Pydantic models** (v2.11+) over unstructured containers for data validation.

## Documentation
- Follow **Google-style docstrings** as specified in the pydocstyle convention.
- Document all public functions, classes, and methods with proper docstrings.
- Include type information in docstrings consistent with annotations.
- Mark TODOs with proper format that can be detected by the TD linting rule.
- Place explanatory comments on separate lines before code, never inline with code.

## API Design
- Use **Pydantic models** for all external input and output validation.
- Ensure **consistent, structured error responses** with clear codes and messages.
- Follow the **Model Context Protocol (MCP)** standards for server implementation.
- Design clean, resource-oriented APIs following RESTful principles.

## Error Handling
- Use the `TRY` linting rules to improve exception handling patterns.
- Catch and handle all **anticipated exceptions** with precise granularity.
- Define **custom exception hierarchies** aligned with the domain model.
- Use `raise from` to **preserve exception traceability** when re-raising exceptions.

## Async Programming
- Properly utilize `asyncio` (v3.4.3+) for I/O-bound operations.
- Follow best practices for asynchronous code in the `api/async_utils.py` module.
- Keep core business logic synchronous unless there's a compelling reason.

## Project Structure
- Maintain clear separation between:
  - `schemas/`: Pydantic models for data validation
  - `api/`: Client implementation and API interfaces
  - `services/`: Business logic and domain services
  - `tools/`: CLI tools and utilities
  
## Code Quality Tools
- Ensure code passes all enabled ruff linting rules including:
  - `E`, `W`: pycodestyle errors and warnings
  - `F`: pyflakes
  - `C`: comprehension improvements
  - `B`: bugbear checks for bug risks
  - `SIM`: code simplification suggestions
  - `ERA`: detection of commented-out code
- Use **black** (v24.2+) for code formatting.
- Run **mypy** (v1.8+) for static type checking.

## GitLab Integration
- Follow the python-gitlab (v5.6+) library patterns and best practices.
- Properly handle GitLab API pagination, authentication, and rate limits.
- Provide clear error messages for GitLab API failures.

## Code Comments
- **Avoid inline comments** - they make code harder to read and maintain.
- Use descriptive variable and function names that make comments unnecessary.
- Comment on "why" code does something, not "what" it does.

## For up-to-date information
- Make tool call to context7 with library id "gitlabhq/gitlabhq"

---
> Source: [Adit-999/gitlab-mcp](https://github.com/Adit-999/gitlab-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-22 -->
