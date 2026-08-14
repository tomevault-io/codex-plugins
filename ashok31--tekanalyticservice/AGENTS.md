# Copilot Instructions — analyticservice

This file defines how GitHub Copilot should behave in Chat and Edit mode for the `analyticservice` repository.

---

## ⚠️ Requirements-First Rule

Before doing ANY work:

1. Read `docs/REQUIREMENTS.md` in full.
2. Confirm the feature has an `Approved` entry before implementing anything.
3. If no entry exists, draft a new requirement file at `docs/requirements/REQ-<next-ID>-<kebab-case-title>.md` and show it to the user before pushing.

> Never write code, open a PR, or push files for a feature without an `Approved` requirement entry.

---

## 🐛 Bug Fixing

For all bug fix tasks, follow **[`docs/BUG-FIXING.md`](../docs/BUG-FIXING.md)**.

Key rules:
- Bug fixes do **not** require a `REQ-NNN` document, but **must** have a Jira ticket (`KN-N`).
- Always branch as `fix/<description>` or `hotfix/<description>` from `main`.
- Write a regression test that fails before the fix and passes after.
- Full test suite must pass and coverage must stay ≥ 80%.
- Squash merge via PR — never push directly to `main`.
- Transition the Jira ticket to `Done` and delete the branch after merge.

---

## Code Style

- Language: JavaScript (Node.js). No TypeScript unless explicitly asked.
- ESLint recommended rules.
- 2 spaces indentation, single quotes, always semicolons, max 100 char line length.
- `async/await` only — no `.then()/.catch()` chains.
- Wrap async handlers in `try/catch`, never expose stack traces to client.
- Parameterized SQL queries only (`$1`, `$2`) — never string-interpolate SQL.
- Use connection pool from `src/db.js` — never create ad-hoc connections.
- `ORDER BY event_time DESC` on all event queries unless specified otherwise.

## Naming
- Variables/Functions: camelCase
- Constants: UPPER_SNAKE_CASE
- Files/Folders: kebab-case
- UI Components: PascalCase

---

## Commits

Follow Conventional Commits:
```
<type>(<scope>): <short summary under 72 chars>
```
Types: `feat`, `fix`, `chore`, `docs`, `test`, `refactor`, `perf`, `breaking`, `ui`

---

## Branching

- Feature: `feature/<description>`
- Fix: `fix/<description>`
- Breaking: `ver<X.Y>`
- Hotfix: `hotfix/<description>`
- UI: `ui/<description>`
- Never push breaking changes directly to `main`.

---

## Pull Requests

Every PR description must include:
- Link to `docs/requirements/REQ-<ID>-<title>.md`
- Jira ticket: `https://srivenkatarama.atlassian.net/browse/KN-<N>`
- Acceptance criteria checklist (each AC from the requirement file)
- NFRs addressed
- Breaking changes
- UI screenshot or Figma link if applicable

---

## Testing

- Jest + Supertest. Storybook for UI.
- 80% minimum code coverage.
- Always mock `src/db.js` in unit tests.
- Test happy path, missing params, invalid input, and DB errors.

---

## Jira

- Use the atlassian MCP tool to fetch Jira issue details.
- Always reference the Jira ticket in PR descriptions.
- Jira project: `KN` at `https://srivenkatarama.atlassian.net`

---

## Design Docs & Confluence

- Design docs go in `docs/design/<YYYY-MM-DD>-<description>.md`.
- Every design doc must be paired with a Confluence page (pageId parent: `7995404`).
- GitHub docs: technical specs, SQL, code decisions.
- Confluence: architecture, user stories, ADRs, runbooks.

---
> Source: [Ashok31/tekanalyticservice](https://github.com/Ashok31/tekanalyticservice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-14 -->
