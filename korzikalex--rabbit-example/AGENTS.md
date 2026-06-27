# GitHub Copilot Instructions

## Project context
This is a small educational RabbitMQ demo for university presentation purposes.

Use Python, FastAPI, RabbitMQ, Docker Compose, uv, and Ruff.

The project has three main runtime parts:
- RabbitMQ broker with the management UI
- backend service built with FastAPI
- worker service that processes queue messages

## General rules
- Before writing or changing code, consult the official documentation first and align with it.
- Prefer the simplest correct implementation.
- Do not introduce hacks, workarounds, or hidden complexity.
- Keep the code easy to explain in a classroom setting.

## Python style
- Follow PEP 8.
- Use snake_case for functions, variables, and module names. :contentReference[oaicite:1]{index=1}
- Use 4 spaces for indentation. :contentReference[oaicite:2]{index=2}
- Keep top-level definitions separated by two blank lines. :contentReference[oaicite:3]{index=3}
- Keep lines readable and consistent with Ruff configuration. Ruff’s default line length is 88 for compatibility with Black and the Ruff formatter, while PEP 8 recommends 79 characters. :contentReference[oaicite:4]{index=4}

## Typing
- Type public functions, request/response models, message payloads, and module boundaries.
- Type important values and interfaces.
- Do not over-type trivial local variables when the type is obvious.
- Prefer clear data models over ad-hoc dictionaries when the structure matters.

## FastAPI rules
- Use FastAPI type hints for request and response data.
- Keep route handlers thin.
- Put business logic in service modules, not inside endpoints.
- Use `async def` only when the operation is actually async I/O. FastAPI supports both sync and async handlers. :contentReference[oaicite:5]{index=5}
- Prefer dependency injection over globals for reusable services and resources. FastAPI provides a dependency injection system. :contentReference[oaicite:6]{index=6}

## RabbitMQ rules
- Follow the official RabbitMQ Python tutorial patterns.
- Use a real AMQP client pattern, with explicit connection and channel setup.
- Declare queues before use in both producer and consumer code so startup order does not matter. The RabbitMQ tutorial explicitly repeats queue declaration for this reason. :contentReference[oaicite:7]{index=7}
- Keep producer, backend, and worker responsibilities clearly separated.
- Message handling should be explicit and predictable.
- Prefer clear naming for exchanges, queues, routing keys, and payloads.
- Keep the demo focused on correctness and clarity, not advanced broker features unless they are needed for the example.

## Docker Compose rules
- Use Docker Compose for local orchestration.
- Provide separate services for RabbitMQ, backend, and worker.
- Use the RabbitMQ management image so the management UI is available.
- Expose the broker on `5672` and the management UI on `15672`. The official RabbitMQ image docs document the management plugin on port `15672`, with broker access on `5672`. :contentReference[oaicite:8]{index=8}
- Prefer environment variables in Compose for configuration, not hardcoded values in application code.
- Keep container startup simple and deterministic.

## uv rules
- Use `pyproject.toml` as the source of truth for dependencies.
- Commit the lockfile.
- Use `uv sync` to create or update the environment and `uv run` to execute project commands in the locked environment. :contentReference[oaicite:9]{index=9}
- Do not manage dependencies with ad hoc `pip install` commands in the project workflow unless explicitly required.
- Keep dependency definitions minimal and explicit.

## Ruff rules
- Keep the code Ruff-clean.
- Prefer formatter-friendly code.
- Avoid suppressing lint rules unless there is a concrete reason.
- When in doubt, write code that is simple, explicit, and mechanically easy to lint and format.

## Testing and reliability
- Add small tests where they improve confidence.
- Prefer unit tests for business logic and integration tests for message flow.
- Keep examples deterministic and easy to reproduce.
- Use explicit error handling for expected failures.

## Writing and naming
- Use descriptive names for modules, functions, queues, and services.
- Avoid abbreviations unless they are standard in messaging or FastAPI contexts.
- Keep comments short and only where they add real value.
- Prefer docstrings for public modules, classes, and functions when the intent is not obvious.

## Implementation preferences
- Prefer composition to inheritance.
- Prefer small modules over large ones.
- Prefer explicit payload schemas over loosely structured dictionaries.
- Prefer straightforward control flow over clever code.
- Prefer stable, boring solutions over experimental ones.

---
> Source: [KorzikAlex/rabbit-example](https://github.com/KorzikAlex/rabbit-example) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-27 -->
