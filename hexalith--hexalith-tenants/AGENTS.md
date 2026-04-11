# Copilot Instructions for Hexalith.Tenants

## Commit Messages

All commit messages **must** follow the [Conventional Commits](https://www.conventionalcommits.org/) specification. This is required for semantic-release to determine version bumps and generate changelogs.

Format: `<type>(<optional scope>): <description>`

- `feat:` -- New feature (triggers **minor** version bump)
- `fix:` -- Bug fix (triggers **patch** version bump)
- `docs:` -- Documentation only
- `refactor:` -- Code change that neither fixes a bug nor adds a feature
- `test:` -- Adding or updating tests
- `chore:` -- Build process, CI, or tooling changes
- `perf:` -- Performance improvement

For breaking changes, add `BREAKING CHANGE:` in the commit body or append `!` after the type (e.g., `feat!:`). This triggers a **major** version bump.

## Code Style

- File-scoped namespaces (`namespace X.Y.Z;`)
- Allman braces (new line before opening brace)
- `_camelCase` private fields
- 4-space indentation
- Warnings as errors
- Nullable enabled globally

## Project Context

- .NET 10 event-sourced microservice built on Hexalith.EventStore
- Uses DAPR for state store, pub/sub, and actors
- 5 NuGet packages: Contracts, Client, Server, Testing, Aspire
- CI/CD: GitHub Actions with semantic-release on merge to main
- Test framework: xUnit + Shouldly + NSubstitute

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Hexalith)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Hexalith)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
