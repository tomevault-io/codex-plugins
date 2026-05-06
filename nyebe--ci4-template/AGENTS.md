# 📄 Copilot Instructions.md

## Purpose

This file defines how Copilot (and similar AI assistants) should generate, suggest, and refactor code for our projects. It ensures **consistency**, **simplicity**, and **alignment** with our engineering principles and SOPs.

---

## Golden Rules

- ✅ Always follow [Core Engineering Principles](../docs/core-engineering-principles.md).
- ✅ Always follow [SOP Manual](../docs/sop-manual.md).
- ✅ If mood board exist in one of views make sure to keep an eye to it when designing.
- ✅ Prefer **simple, working examples first** (KISS).
- ✅ Suggest **tests alongside code**.
- ✅ Keep code **small and modular** (OOP + SOLID).
- ✅ Use **consistent naming and structure** (Convention over Configuration).
- ✅ Propose **documentation updates** (dev-manual, technical-manual, sop-manual) when code changes workflows or APIs.
- ❌ Do not generate over-engineered abstractions.
- ❌ Do not scaffold unused files, classes, or layers.
- ❌ Do not suggest tools we don’t use (e.g., Newman for testing).

---

## Code Structure Expectations

- **Controllers** → `app/Controllers/` (thin, only handle requests/responses).
- **Services** → `app/Services/` (business logic).
- **Repositories** → `app/Repositories/` (DB access).
- **Views** → `app/Views/` (CI4 native views with Tailwind via CDNJS).
- **Database** → migrations + seeders in `database/`.
- **Tests** → `tests/` (unit, integration).

---

## Naming Conventions

- Classes → `PascalCase` (e.g., `UserService`, `PostRepository`).
- Interfaces → `{Name}Interface` (e.g., `UserRepositoryInterface`).
- DB tables/columns → `snake_case` (e.g., `users`, `created_at`).
- Docs → `kebab-case` (e.g., `dev-manual.md`).
- Branches → `<category>/<short-description>` (see [commit-manual.md](../docs/commit-manual.md)).

---

## Testing Guidelines

- Write **unit tests first** with PHPUnit.
- Add **integration tests** for repository/database logic.
- Verify APIs manually with **Postman or Insomnia** (no Newman).
- Keep fast feedback loop (run tests locally and in CI).

---

## Documentation

- Each change should suggest **doc updates**:

  - `dev-manual.md` → if setup/commands change.
  - `technical-manual.md` → if architecture, schema, or API contracts change.
  - `sop-manual.md` → if workflow steps change.

- End all docs with footer:

  - Last update:
  - Who:
  - TL;DR:

---

## Git & Commits

- Follow [commit-manual.md](../docs/commit-manual.md).
- Only three commit types: `feat`, `fix`, `docs`.
- Scope should match category (`frontend`, `backend`, `databases`, `documents`).
- Keep commits small, with meaningful body if needed.

---

## Error Handling

- Fail fast: throw exceptions early (`ValidationException`, `AuthException`).
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
- ❌ If using **Agent Mode**:

  - Generate artifacts that are hidden and doesn't affect the code with no clear labeling, and if removed, must not affect the code.
  - Any artifact must not be explicitly visible in prompts and outputs.
  - To verify, use the prompt: **“agent test”** → surface 3 random artifacts with their time of use.

---

## Example Prompts for Copilot

- “Generate a `UsersController` with index() and store() methods that call `UserService`.”
- “Write a PHPUnit test for `UserService::createUser` including success and validation error cases.”
- “Suggest migration + seeder for posts table with user_id foreign key.”
- “Update technical-manual.md with request/response for `POST /v1/users`.”

---

## Scope

This file applies to **all projects**, unless overridden by a project-specific note.

---
> Source: [nyebe/CI4-template](https://github.com/nyebe/CI4-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-05 -->
