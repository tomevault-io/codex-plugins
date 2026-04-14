# Copilot Instructions for Summer-Chill

## Project Context
- This repository is a C# .NET worker service that integrates with FGLair APIs to monitor and control climate devices.
- Prioritize production safety, reliability, and maintainability over cleverness.
- Keep changes minimal, focused, and aligned with existing code style.

## Core Priorities
- Preserve behavior unless a change request explicitly requires behavior changes.
- Prefer clear, explicit code over terse or overly abstract code.
- Keep methods focused and practical; split only when it improves readability and testability.
- Add comments only when logic is non-obvious.

## C# Naming Conventions
- Types, methods, properties, events, and constants: `PascalCase`.
- Interfaces: `I` prefix + `PascalCase` (for example: `IFGLairClient`).
- Private fields: `_camelCase`; static private fields: `s_camelCase`; thread-static fields: `t_camelCase`.
- Parameters and locals: `camelCase`.
- Async methods: `Async` suffix and return `Task` or `Task<T>`.
- Avoid unclear abbreviations unless they are domain-standard terms.

## Null Safety and Validation
- Nullable reference types are enabled; treat nullability warnings as important design feedback.
- Annotate optional references with `?` and validate required inputs early.
- Use `ArgumentNullException.ThrowIfNull(...)` for required dependencies/arguments.
- Prefer safe checks (`is not null`, pattern matching) before dereferencing maybe-null values.
- Use the null-forgiving operator (`!`) only when the non-null guarantee is explicit and justified.

## Async, Cancellation, and Concurrency
- Use `async`/`await` end-to-end for I/O operations.
- Never block async code with `.Result` or `.Wait()`.
- `async void` is only for event handlers.
- Pass `CancellationToken` through async call chains, especially in worker loops and HTTP calls.
- Avoid tight loops; use `PeriodicTimer` or token-aware delays for recurring work.
- Use bounded retries for transient failures. Avoid infinite retries.
- For concurrent async work, prefer `Task.WhenAll(...)` with materialized tasks when needed.

## Error Handling and Logging
- Catch specific exceptions where recovery is meaningful; do not hide faults with overly broad catches.
- Log with structured logging and useful context keys.
- Never log secrets or sensitive data: passwords, access tokens, refresh tokens, usernames, or full auth payloads.
- Validate configuration early and fail fast with actionable error messages.

## Dependency Injection and Composition
- Prefer constructor injection for dependencies.
- Depend on interfaces for external services and boundaries.
- Keep domain and API logic separated from hosting/bootstrap code.
- Avoid static mutable state when a scoped or singleton service is more testable.

## Modern C# Style
- Use records for immutable DTO-like data where appropriate.
- Prefer pattern matching and switch expressions for clear branching logic.
- Use object/collection initializers and collection expressions when they improve readability.
- Use `var` when the type is obvious from the right-hand side; otherwise use explicit types.
- Use raw string literals for multi-line JSON or patterns when they improve clarity.
- Use `required` and `init` for models that must be initialized once and then remain stable.

## Testing and Validation
- Build and validate affected code paths after making changes.
- If a test project exists, add or update tests with AAA structure (Arrange, Act, Assert).
- For async code, write async tests that return `Task`.
- If no tests are available, include a short manual verification checklist in the change summary.

## Configuration and Security
- Keep runtime configuration in `appsettings.template.json`, environment variables, and user secrets.
- Do not hardcode credentials, API keys, device identifiers, or environment-specific endpoints.
- Document any new configuration keys and expected defaults.

## Scope and Diffs
- Prefer small, focused patches over broad refactors.
- Avoid unrelated formatting or stylistic churn in untouched files.
- Keep public APIs stable unless a breaking change is explicitly requested.
- When requirements are ambiguous, clarify intent before introducing behavioral changes.

## Copilot Working Rules for This Repo
- Follow existing project conventions before introducing new patterns.
- Be defensive around API errors, network instability, and null responses.
- Respect cancellation and shutdown behavior in all background processing.
- Keep logs actionable and safe for production environments.
- Favor maintainable code that a teammate can understand quickly.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AlexandreBrisebois) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
