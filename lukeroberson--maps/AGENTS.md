# Project Overview

An app for creating printable/exportable maps, with a browser based UI.

## Features

* A separate frontend and backend
    * The frontend is in the `frontend` directory
    * The backend is the API, which is in the `api` directory
    * Each of these has a `docs` folder containing documentation
* Map data comes from OpenStreetMap
* Projects are created
    * Region maps are defined at the root of the project (multiple regions can exist per project)
        * Boundaries for each region map are defined
    * Under each region map, suburb maps are created within the region maps boundaries
        * Once defined, these are separately editable
    * Under suburb maps, smaller maps are created with their own boundaries
* Each map has editable layers for annotations


# Architecture

Preferred languages and libraries are:
* Python for back end code
* Flask, for web backend
* TypeScript for front end code
* React for frontend UI
* SQLite for data storage

Deployment:
* During testing, this runs locally as a Flask application
* In production, this is built as a container and includes WSGI


---
description: 'Python coding conventions and guidelines'
applyTo: '**/*.py'
---

# Python Coding Conventions

## Python Instructions

- Write clear and concise comments for each function.
- Ensure functions have descriptive names and include type hints.
- Provide docstrings following PEP 257 conventions.
- Use the `typing` module for type annotations (e.g., `List[str]`, `Dict[str, int]`).
- Prefer classes to standalone functions whenever appropriate
- Lines should not contain only tabs or spaces. Each line should contain code or be completely blank.

## General Instructions

- Always prioritize readability and clarity.
- Write code with good maintainability practices, including comments on why certain design decisions were made.
- Use consistent naming conventions and follow language-specific best practices.
- Write concise, efficient, and idiomatic code that is also easily understandable.

## Code Style and Formatting

- Follow the **PEP 8** style guide for Python.
- Ensure lines do not exceed 79 characters.
- For functions and methods, each argument should be on a separate line

## Example of Function Formatting

```python
def calculate_area(
    radius: float
) -> float:
    """
    Calculate the area of a circle given the radius.
    
    Parameters:
    radius (float): The radius of the circle.
    
    Returns:
    float: The area of the circle, calculated as π * radius^2.
    """

    import math
    return math.pi * radius ** 2
```

## Example of Class Formatting

```python
class SampleClass:
    """
    A sample class used to show docstrings and code formatting

    Attributes:
        config (str): A string of configuration information

    Methods:
        __init__:
            Initialize QueryProcessor
        process:
            Process a user query
    """

    def __init__(
        self,
        config: str,
    ) -> None:
        """
        Initialize the class

        Args:
            config (str): Configuration

        Returns:
            None
        """

        self.config = config

    async def process(
        self,
        user_query: str
    ) -> None:
        """
        Process a user query.

        Args:
            user_query (str): The user's query

        Returns:
            None
        """

        try:
            print("This is where code is added")

        except Exception as e:
            print(f"❌ A problem has occured: {e}\n")

```


---
description: 'Guidelines for TypeScript Development targeting TypeScript 5.x and ES2022 output'
applyTo: '**/*.ts'
---

# TypeScript Development

## Core Intent

- Respect the existing architecture and coding standards.
- Prefer readable, explicit solutions over clever shortcuts.
- Extend current abstractions before inventing new ones.
- Prioritize maintainability and clarity, short methods and classes, clean code.

## General Guardrails

- Target TypeScript 5.x / ES2022 and prefer native features over polyfills.
- Use pure ES modules; never emit `require`, `module.exports`, or CommonJS helpers.
- Rely on the project's build, lint, and test scripts unless asked otherwise.
- Note design trade-offs when intent is not obvious.

## Project Organization

- Follow the repository's folder and responsibility layout for new code.
- Use kebab-case filenames (e.g., `user-session.ts`, `data-service.ts`) unless told otherwise.
- Keep tests, types, and helpers near their implementation when it aids discovery.
- Reuse or extend shared utilities before adding new ones.

## Naming & Style

- Use PascalCase for classes, interfaces, enums, and type aliases; camelCase for everything else.
- Skip interface prefixes like `I`; rely on descriptive names.
- Name things for their behavior or domain meaning, not implementation.

## Formatting & Style

- Run the repository's lint/format scripts (e.g., `npm run lint`) before submitting.
- Match the project's indentation, quote style, and trailing comma rules.
- Keep functions focused; extract helpers when logic branches grow.
- Favor immutable data and pure functions when practical.

## Type System Expectations

- Avoid `any` (implicit or explicit); prefer `unknown` plus narrowing.
- Use discriminated unions for realtime events and state machines.
- Centralize shared contracts instead of duplicating shapes.

## Async, Events & Error Handling

- Use `async/await`; wrap awaits in try/catch with structured errors.
- Guard edge cases early to avoid deep nesting.
- Send errors through the project's logging/telemetry utilities.
- Surface user-facing errors via the repository's notification pattern.
- Debounce configuration-driven updates and dispose resources deterministically.

## Architecture & Patterns

- Follow the repository's dependency injection or composition pattern; keep modules single-purpose.
- Observe existing initialization and disposal sequences when wiring into lifecycles.
- Keep transport, domain, and presentation layers decoupled with clear interfaces.
- Supply lifecycle hooks (e.g., `initialize`, `dispose`) and targeted tests when adding services.

## External Integrations

- Instantiate clients outside hot paths and inject them for testability.
- Never hardcode secrets; load them from secure sources.
- Apply retries, backoff, and cancellation to network or IO calls.
- Normalize external responses and map errors to domain shapes.

## Security Practices

- Validate and sanitize external input with schema validators or type guards.
- Avoid dynamic code execution and untrusted template rendering.
- Encode untrusted content before rendering HTML; use framework escaping or trusted types.
- Use parameterized queries or prepared statements to block injection.
- Keep secrets in secure storage, rotate them regularly, and request least-privilege scopes.
- Favor immutable flows and defensive copies for sensitive data.
- Use vetted crypto libraries only.
- Patch dependencies promptly and monitor advisories.

## Configuration & Secrets

- Reach configuration through shared helpers and validate with schemas or dedicated validators.
- Handle secrets via the project's secure storage; guard `undefined` and error states.
- Document new configuration keys and update related tests.

## UI & UX Components

- Sanitize user or external content before rendering.
- Keep UI layers thin; push heavy logic to services or state managers.
- Use messaging or events to decouple UI from business logic.

## Testing Expectations

- Add or update unit tests with the project's framework and naming style.
- Expand integration or end-to-end suites when behavior crosses modules or platform APIs.
- Run targeted test scripts for quick feedback before submitting.
- Avoid brittle timing assertions; prefer fake timers or injected clocks.

## Performance & Reliability

- Lazy-load heavy dependencies and dispose them when done.
- Defer expensive work until users need it.
- Batch or debounce high-frequency events to reduce thrash.
- Track resource lifetimes to prevent leaks.

## Documentation & Comments

- Add JSDoc to public APIs; include `@remarks` or `@example` when helpful.
- Write comments that capture intent, and remove stale notes during refactors.
- Update architecture or design docs when introducing significant patterns.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LukeRoberson)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/LukeRoberson)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
