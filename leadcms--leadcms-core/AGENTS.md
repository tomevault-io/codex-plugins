# GitHub Copilot Instructions

## Migrations

- Never create or edit EF Core migration files manually.
- Only add migrations by running `dotnet ef migrations add` as part of the implementation, or let a developer do it.

## Development Guidelines

### Test-Driven Development (TDD)

- **Test-First**: Always write failing tests before implementing features or fixes
- **Bug Fix Protocol**: Reproduce bug with test → fix → verify
- **Coverage Goal**: Improve from current 60% to >80%
- **Test Data**: Minimal, generic, privacy-compliant test data only

### Test Execution

- **Never run the full test suite** — it takes too long (700+ tests, ~7 minutes). Only run the specific test classes or test methods that you believe could be affected by the current changes.
- Let the developer run the complete suite when they decide it is appropriate.

### Error Handling in Controllers

- **Never use try/catch in controllers.** All exceptions are intercepted and handled centrally by `ErrorsController` (`/error` endpoint via the ASP.NET exception handler middleware).
- If a new exception type needs specific HTTP status mapping, add a `case` for it in `ErrorsController.HandleError()` rather than catching it in individual controller actions.
- This keeps controllers thin and avoids duplicated error-handling boilerplate across the solution.

---
> Source: [LeadCMS/leadcms.core](https://github.com/LeadCMS/leadcms.core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-24 -->
