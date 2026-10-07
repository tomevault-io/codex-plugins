# GitHub Copilot Instructions

Use this repository as a starter template for modern ASP.NET Core and C# services.

## Follow These Rules

- Preserve Clean Architecture boundaries.
- Keep `Domain` free of framework, EF Core, Azure SDK, and HTTP concerns.
- Keep `Application` focused on use cases, contracts, validation, and orchestration.
- Keep `Infrastructure` focused on EF Core, external integrations, and implementation details.
- Keep `WebApi` thin and transport-oriented.

## Coding Defaults

- Prefer explicit, readable C# over clever shortcuts.
- Use constructor injection.
- Keep nullable reference types enabled.
- Use async/await correctly and propagate cancellation tokens.
- Use strongly typed options for configuration.
- Use DTOs for HTTP contracts.
- Use `ProblemDetails` for failures.

## Data and Persistence

- Use EF Core migrations instead of manual schema changes.
- Keep mappings in configuration classes.
- Bound pagination and collection queries.
- Add concurrency handling where needed.
- Introduce repositories only when they clarify the application layer.

## Testing

- Add tests for positive and negative scenarios.
- Use xUnit, FluentAssertions, and NSubstitute.
- Use integration tests for real HTTP and persistence behavior when appropriate.

## Documentation and Verification

- Keep README and prompt assets aligned with code changes.
- Prefer minimal, maintainable changes over broad rewrites.
- Verify build and test impact when making non-trivial changes.

## Extra Prompt Assets

For deeper guidance, consult:

- `AGENTS.md`
- `CLAUDE.md`
- `.claude/agents/`
- `.claude/skills/`
- `prompts/README.md`

---
> Source: [DIL8654/claude-code-dotnet-template](https://github.com/DIL8654/claude-code-dotnet-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-10-07 -->
