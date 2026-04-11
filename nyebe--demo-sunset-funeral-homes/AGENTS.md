# 📄 Copilot Instructions

- ✅ Always follow [Core Engineering Principles](../docs/core-engineering-principles.md).
- ✅ Always follow [SOP Manual](../docs/sop-manual.md).
- ✅ Always follow [Branching guide](../docs/branching-guidelines.md) and [Commit Manual](../docs/commit-manual.md).
- ✅ If a mood board exists in views, keep designs aligned with it.
- ✅ Prefer simple, working examples first (KISS).
- ✅ Suggest tests alongside code.
- ✅ Keep code small and modular (OOP + SOLID).
- ✅ Use consistent naming and structure (Convention over Configuration).
- ✅ Propose documentation updates (dev-manual, technical-manual, sop-manual) when code changes workflows or APIs.
- ❌ Do not generate over-engineered abstractions.
- ❌ Do not scaffold unused files, classes, or layers.
- ❌ Do not suggest tools we don't use (e.g., Newman for testing).
- ❌ Do not use single-letter or three-letter names for variables, functions, or classes; prefer readable names (e.g., avoid $e, use $error).
- ❌ Do not perform unnecessary conversions between arrays and objects; use the data structure as-is when possible.
- ❌ Do not add features or code that are not explicitly mentioned, you can suggest and wait for user approval

## Purpose

This file defines how Copilot (and similar AI assistants) should generate, suggest, and refactor code for our projects. It ensures consistency, simplicity, and alignment with our engineering principles and SOPs.

---

## Code Structure Expectations

- Controllers → app/Controllers/ (thin; handle requests/responses only).
- Services → app/Services/ (business logic).
- Repositories → app/Repositories/ (DB access).
- Views → app/Views/ (CI4 native views with Tailwind via CDN).
- Database → migrations + seeders (CI4 default: app/Database/).
- Tests → tests/ (unit, integration).
- Public assets → public/ (css, js, images).

Keep controllers thin; push logic into Services/Repositories.

---

## View Documentation

All view files must include a standardized comment header with:

- Page/Component: Full path to the view file
- Purpose: Brief description of what the view does
- Data Contract: List of expected variables with types and descriptions

Example for a page:

```
// Page: user/landing.php
// Purpose: Landing page for the funeral home website
// Data Contract:
// - $services: object array | string | null - Services data for carousel
```

Example for a component:

```
// Component: components/sections/cta.php
// Purpose: Call-to-action section with heading, subtext, and buttons
// Data Contract:
// - $heading: string|null - Main heading text
// - $primary: array - Primary button config with 'label' and 'href'
```

---

## Naming Conventions

- Classes → PascalCase (e.g., UserService, PostRepository).
- Interfaces → {Name}Interface (e.g., UserRepositoryInterface).
- DB tables/columns → snake_case (e.g., users, created_at).
- Docs → kebab-case (e.g., dev-manual.md).
- Branches → <category>/<short-description> (see [branching-guidelines.md](../docs/branching-guidelines.md)).

Categories: frontend, backend, databases, documents.
Examples: backend/jwt-auth-service, documents/update-sop-manual.

---

## Testing Guidelines

- Write unit tests first with PHPUnit.
- Add integration tests for repository/database logic.
- Verify APIs manually with Postman or Insomnia (no Newman).
- Keep the feedback loop fast (run tests locally and in CI).

---

## Documentation

Suggest doc updates with every meaningful change:

- dev-manual.md → if setup/commands change.
- technical-manual.md → if architecture, schema, or API contracts change.
- sop-manual.md → if workflow steps change.
- branching-guidelines.md and commit-manual.md → if Git flow changes.

End all docs with a footer:

- Last update:
- Who:
- TL;DR:

---

## Git & Commits

- Only three commit types: feat, fix, docs.
- Scope should match the branch category (frontend, backend, databases, documents).
- Format:
  - <type>(<scope>): <short summary>
  - Optional body explains why + what changed (wrap at ~72 chars).
- Branch names follow <category>/<short-description>; see [branching-guidelines.md](../docs/branching-guidelines.md) for workflow and PR rules.
- See [commit-manual.md](../docs/commit-manual.md) for examples.

---

## Error Handling

- Fail fast: validate early and throw clear exceptions (e.g., ValidationException, AuthException).
- Return consistent error JSON:
  ```json
  {
    "error": {
      "code": "VALIDATION_ERROR",
      "message": "...",
      "details": []
    }
  }
  ```
- Always log context: request ID, user ID, route.

---

## Prohibited / Restricted Behaviors

- ❌ Do not put raw SQL or business logic inside controllers.
- ❌ Do not bypass services when accessing repositories.
- ❌ Do not introduce hidden global state or magic methods.
- ❌ Do not create artifacts that cannot be easily identified or explained.
- If using Agent Mode:
  - ❌ Do not generate hidden or unlabeled artifacts.
  - ✅ Any artifact must be explicit, documented, and removable without side effects.
  - For verification, respond to “agent test” by surfacing 3 random artifacts with their time of use.

---

## Example Prompts for Copilot

- “Generate a UsersController with index() and store() that call UserService.”
- “Write a PHPUnit test for UserService::create including success and validation error cases.”
- “Suggest migration + seeder for services table with slug and cost fields.”
- "Update technical-manual.md with request/response for POST /v1/services and link branching-guidelines.md."

---

## Scope

This file applies to all projects, unless overridden by a project-specific note.

---

Last update: 2025-10-17
Who: AI Assistant
TL;DR: Keep it simple, follow CI4 layering (Controllers → Services → Repositories), use branches like category/short-description, commits as feat|fix|docs(scope): summary, write tests, update docs with footers, and document all views with Page/Component/Purpose/Data Contract headers.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nyebe)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/nyebe)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
