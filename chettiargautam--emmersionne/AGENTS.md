# Immersion repository-wide Copilot instructions

## Operating mode
- Read the surrounding code before writing or changing code.
- Prefer modifying existing files over creating new files.
- Do not create new files unless they are clearly necessary for cohesion, maintainability, separation of concerns, or testability.
- Before adding a file, identify the most appropriate existing module or folder and verify whether the change belongs there instead.
- Before deleting, renaming, or moving anything, verify imports, references, routes, tests, and runtime usage.
- After completing a change, remove stale code, dead branches, unused imports, unused exports, and obsolete files introduced or exposed by the change.

## Design philosophy
- Prefer correct, scalable, production-quality solutions over patches or temporary fixes.
- Optimize for clear system design, strong boundaries, maintainability, and readability.
- Keep the number of files reasonable, but do not force everything into oversized files.
- Keep modules focused and cohesive.
- Reuse existing patterns, schemas, services, hooks, stores, and UI primitives before inventing new ones.
- Avoid hidden coupling, implicit state, and scattered hard-coded logic.

## Architecture
- Preserve clean boundaries between routes, schemas, services, persistence, configuration, and presentation layers.
- Use classes for stateful services, orchestration, adapters, repositories, and domain objects that benefit from encapsulation.
- Use plain functions for pure transformations, parsing, formatting, validation helpers, and stateless utilities.
- Prefer explicit interfaces, typed contracts, and clear dependency flow.
- Do not pass large unstructured bags of data through many layers when a typed object or schema would be clearer.

## Documentation and readability
- Prefer docstrings, JSDoc, types, and clear naming over inline comments.
- Avoid casual comments, decorative comments, redundant comments, and commented-out code.
- Only add comments when documenting invariants, non-obvious constraints, tricky external behavior, or necessary temporary workarounds.
- Python modules must begin with a file docstring describing the module purpose, major responsibilities, and important dependencies.
- Public Python classes and public Python functions must have docstrings.
- Exported TypeScript functions, custom hooks, and complex components should have JSDoc when behavior is non-trivial.

## Naming and formatting
- Match existing project conventions first when they are already consistent.
- Prefer descriptive names over short or vague names.
- Python files, functions, methods, and variables use snake_case.
- Python classes use PascalCase.
- TypeScript variables and functions use camelCase.
- React components, types, interfaces, and enums use PascalCase.
- Keep imports clean, grouped logically, and free of unused entries.
- Maintain consistent formatting and spacing across the repository.

## Configuration and constants
- Never hard-code secrets, tokens, API keys, or environment-specific endpoints in feature code.
- Never scatter repeated numeric literals, retry counts, timeout values, default settings, route strings, theme values, or preference defaults throughout the codebase.
- Use a clear, centralized configuration strategy for backend and frontend code each separately.
- Backend runtime configuration should live in a dedicated typed settings or configuration module.
- Backend non-secret reusable defaults and application-wide constants should live in a dedicated constants module.
- Frontend environment access should be centralized in a dedicated configuration module.
- Frontend non-secret constants, route strings, UI defaults, font identifiers, font sizes, spacing tokens, color tokens, and theme values should be centralized in dedicated constants or theme modules.
- If a value is reused, environment-specific, security-sensitive, likely to change, or important for global behavior, it must not remain inline in feature code.
- Do not duplicate constants in multiple places. Reuse the canonical definition.

## Backend quality
- Keep route handlers thin.
- Put business logic into services or other appropriate domain modules.
- Prefer typed request and response contracts.
- Handle errors explicitly and return meaningful messages.
- Use retries, timeouts, backoff, and cancellation handling where external or asynchronous operations can fail.
- Avoid blocking work in async flows.
- Design for observability and future debugging.

## Frontend quality
- Prefer clear user-visible loading, success, empty, and failure states.
- Do not leave users uncertain about whether work is in progress.
- Prefer resilient async flows with retry and recovery behavior when appropriate.
- Keep page-level components thin when logic can be moved into hooks, stores, API clients, or focused reusable components.
- Keep styling consistent with shared tokens and theme rules.

## Testing
- New backend behavior should include tests.
- Changed backend flows should update or add endpoint tests and service-level tests as appropriate.
- Important multi-step flows should have orchestration or end-to-end coverage where feasible.
- Frontend behavior should be testable and should cover visible state transitions and user interactions.
- Remove stale tests that no longer reflect real behavior.

## Cleanup discipline
- When asked to clean up the codebase, aggressively identify and remove dead files, dead functions, dead classes, dead variables, dead exports, dead imports, stale dependencies, obsolete tests, and commented-out code.
- Do not leave backup files, duplicate implementations, or abandoned experimental files in the repository.
- Ensure the remaining code follows the repository rules after cleanup.

## Response behavior
- When implementing a change, first inspect relevant files and summarize the intended touch points before broad edits.
- When uncertain about architecture, math, inference logic, or ambiguous product requirements, ask focused clarifying questions rather than making fragile assumptions.
- When library behavior, framework behavior, or best practices are uncertain or likely to matter, use web research instead of guessing.
- Do not hallucinate files, functions, routes, or conventions that do not exist in the repository.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chettiargautam)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/chettiargautam)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
