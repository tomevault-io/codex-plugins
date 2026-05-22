# Engineering Standards for Senior Python Development

These standards define how Python code should be generated across this repository to ensure readability, reliability, and production readiness.

---

## 1. Core Principles

### Clarity and Maintainability
- Prefer explicit, intention-revealing code over cleverness.
- Optimize for readability first; optimize performance only when there is evidence of a bottleneck.
- Keep functions/classes focused on a single responsibility.

### Production Readiness
- Code must include robust error handling and observable behavior.
- Runtime behavior should be debuggable through structured logging.
- Interfaces must be type-safe and static-analysis friendly.

### Documentation Discipline
- Every public module, class, and function must include a concise docstring covering:
  - Short purpose
  - key side effects (if any)
  - high-level error behavior (if relevant)

---

## 2. Python Language and Style

### Type Hints (Required)
- Fully type all function signatures, method signatures, and class attributes.
- Use Python 3.10+ syntax (`str | None`, `list[str]`, `dict[str, int]`).
- Keep code compatible with `mypy`/`pyright` expectations.

### Preferred Language Features
- Use `dataclasses` for simple immutable domain containers when appropriate.
- Use `match` when it improves clarity.
- Use context managers for resources (files, sessions, locks, network clients).

### Strings and Filesystem
- Use `pathlib.Path` for filesystem operations.
- Use f-strings for regular string interpolation.

### Imports

- Follow PEP 8 import order: standard library, third-party, then local/project imports, with one blank line between groups.
- **Placement:** Place all imports at the top of the file immediately after the module docstring and before constants, classes, or functions.
- **Grouping:** Group imports in three sections with one blank line between groups: standard library, third-party, local/project imports.
- **Ordering:** Sort imports within each group alphabetically. Use `isort` to enforce automatically.
- **Style:** Prefer absolute imports. Avoid wildcard imports (`from module import *`) and avoid deep relative imports when possible.
- **Unused & Linting:** Remove unused imports. Enforce with `ruff`/`flake8`/`pylint` in CI.
- **Type-only imports:** For imports used only for typing that are expensive or cause cycles, use `if TYPE_CHECKING:` or `from __future__ import annotations` and `typing.TYPE_CHECKING`.
- **Conditional imports:** When importing conditionally (platform, optional dependency), add a short comment explaining why.
- **Shared models:** Import shared models at module level.
- **No lazy-import helpers:** Do not use lazy-import helper functions for shared models. Only defer imports when required to avoid a verified import cycle or an expensive optional dependency, and document the reason briefly.
- **Example:**

```
# module docstring

import logging
import math

from fastapi import FastAPI
from starlette.requests import Request

from myproject.utils import helper_function
```

### Logging (Mandatory, Non-Negotiable)
**Hard Rule:** All runtime logging must use Python's standard library `logging` module.

- Use module-level loggers only:
  - `logger = logging.getLogger(__name__)`
- Do not use `print()` for runtime diagnostics.
- Do not use third-party logging libraries (e.g., `loguru`) unless explicitly requested.
- Log call message templates must be literal strings.
- Use `%`-style parameterized logging for dynamic values:
  - `logger.info("Processed %s records", count)`
- Never use f-strings in logging calls.
- Log failures with context using `logger.exception(...)`.

### String Formatting Precedence
- Use f-strings for non-logging strings.
- Logging calls are the explicit exception and must use `%`-style placeholders.
- If any conflict exists, logging rules take precedence for all `logger.*(...)` calls.

### Global State
- Avoid mutable module-level state.
- Constants must be uppercase and typed.

---

## 3. Data Modeling and Validation

### Default Modeling Strategy (Mandatory)
**Hard Rule:** Prefer `@dataclass(frozen=True)` for application/domain configuration and internal data structures.

**Modeling Precedence Rule:** If both dataclass and Pydantic are possible, choose `@dataclass(frozen=True)` unless boundary parsing/serialization explicitly requires Pydantic.

- Use dataclasses as the default for:
  - internal DTOs
  - domain models
  - settings/config objects (unless a framework requires otherwise)
- Prefer immutability:
  - `@dataclass(frozen=True)`
- Validate invariants in `__post_init__`.
- Keep validation deterministic and side-effect free.
- Do not place business logic in model validation methods.

### When Pydantic Is Allowed
- Use `pydantic.BaseModel` only at external boundaries where schema parsing/serialization is required:
  - FastAPI request/response models
  - external payload parsing
- If Pydantic is used, prefer strict mode and explicit schemas:
  - `model_config = ConfigDict(strict=True, extra="forbid")`
- Do not use Pydantic for internal domain models by default.
- For settings/config objects, use dataclasses unless framework integration explicitly requires Pydantic settings.

### Error Transparency
- Never swallow validation errors.
- Propagate or log with full context.

---

## 4. Architecture and Design

### Dependency Injection
- Pass external dependencies explicitly (DB sessions, HTTP clients, caches, brokers).
- Avoid hidden globals and implicit singletons.

### Purity and Side Effects
- Keep business logic deterministic and testable.
- Isolate I/O side effects at boundaries.

### Async-First
- Use async for I/O-bound operations.
- FastAPI endpoints should be async unless a specific constraint requires sync.

### Interface Segregation
- Keep interfaces narrow and domain-oriented.
- Avoid multi-purpose god classes.

### Error Boundaries
- Define where errors are caught, transformed, logged, and rethrown.
- Do not leak infrastructure exceptions into domain layers without translation.

---

## 5. Testing Standards

### Framework and Structure
- Use `pytest`.
- Use fixtures for setup/teardown and dependency injection.
- Use parametrization for boundary and edge-case coverage.

### Database Testing
- Use in-memory SQLite (or `StaticPool`) for SQLAlchemy tests.
- Never connect tests to persistent/production databases.
- Isolate DB session state per test; close or rollback reliably.

### Contract Testing
- Verify FastAPI behavior for:
  - request/response schemas
  - status codes
  - error shapes
  - serialization behavior

### Coverage Policy
- Aim for high coverage on critical paths.
- Prioritize meaningful tests over raw percentage targets.

---

## 6. Security and Performance

### Security Basics
- Never hardcode credentials or secrets.
- Load secrets from environment/config providers.
- Validate and sanitize all external inputs.
- Reject unexpected fields by default.

### Safe Defaults
- Prefer fail-closed behavior.
- Validate assumptions explicitly.

### Performance Pragmatism
- Use generators/streaming for large payloads.
- Add caching (`lru_cache` or external cache) for deterministic expensive operations where justified.

---

## 7. Decision-Making Expectations

### Clarify Uncertainty
- Ask for missing architectural context rather than guessing.

### Communicate Trade-offs
- When multiple valid options exist, present concise pros/cons.

### No Fabrication
- Do not invent APIs or library behavior.
- If version-specific behavior is unknown, state assumptions explicitly.

---

## 8. Completion Criteria

A task is considered complete only when all are true:
1. Code is syntactically correct and follows these standards.
2. Public modules/classes/functions include meaningful docstrings.
3. Logging and error handling are appropriate and consistent.
4. Type hints are present and coherent.
5. Relevant tests pass.
6. Data modeling policy is respected: `@dataclass(frozen=True)` by default, `pydantic.BaseModel` only at explicit boundaries.
7. Security expectations are met (no secret leakage, proper validation).
8. Code is ready for peer review/approval.

---
> Source: [lersam/machine-learning](https://github.com/lersam/machine-learning) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-22 -->
