# Copilot Instructions for This Repository

You are assisting with production code in this repository.

## General standards
- Prefer existing patterns, utilities, services, hooks, and shared components before introducing new ones.
- Match the repository’s naming, structure, typing, and error-handling conventions.
- Keep code simple, readable, and maintainable.
- Prefer explicit implementations over clever or overly generic abstractions.
- Do not introduce speculative extensibility.

## Implementation rules
- Do not invent APIs, imports, methods, config values, or types.
- Do not generate placeholder logic, fake implementations, or unfinished production code.
- Validate inputs and handle failure cases explicitly.
- Reuse existing domain models, helpers, and test utilities where possible.
- Keep changes focused and minimal.

## Architecture rules
- Keep business logic in the appropriate service/domain layer.
- Keep data access concerns inside the proper data/repository layer.
- Do not duplicate utilities or patterns that already exist in the codebase.
- Follow existing boundaries between UI, application, domain, and data layers.

## Testing rules
- Add or update tests for any new or changed behavior.
- Cover happy paths, edge cases, and failure cases.
- Prefer behavior-focused tests over implementation-detail tests.
- Reuse existing test helpers, fixtures, and patterns.

## Safety rules
- Never hardcode secrets, tokens, API keys, or environment-specific values.
- Be conservative with authentication, authorization, billing, migrations, and security-sensitive code.
- When requirements are unclear, do not guess business logic. Ask for clarification or state assumptions clearly.

## Comments and documentation
- Do not add comments that merely restate the code.
- Write comments only when they clarify intent, constraints, or tradeoffs.
- Keep generated documentation concise and accurate.

## Code Quality Rules
- Avoid unnecessary wrappers, helper functions, and abstractions.
- Do not refactor working code unless there is a clear benefit.
- Prefer small, reviewable changes over broad rewrites.
- Do not produce generic template-style code that ignores repository patterns.
- Generated code must be understandable, testable, and consistent with the surrounding code.

# PR review instructions
When reviewing pull requests:

- Prioritize correctness, security, and maintainability over style nits.
- Flag unnecessary abstractions, wrappers, and speculative extensibility.
- Check whether existing utilities or patterns in the repo should be reused.
- Flag missing input validation, missing failure-path handling, and weak tests.
- Be strict on auth, billing, migrations, secrets, infra, and data-access changes.
- Prefer small focused changes. Call out PRs that are too broad to review safely.
- When logic changes, look for matching test coverage.
- Do not approve code that appears generated but is poorly explained or weakly validated.

---
> Source: [Lannzo/job-tracker](https://github.com/Lannzo/job-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-28 -->
