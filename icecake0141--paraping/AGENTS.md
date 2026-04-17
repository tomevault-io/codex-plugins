# Policy: LLM-Generated Code & Pull Request Requirements

Purpose
- Ensure any code produced or modified with the assistance of an LLM includes required license information and explicit LLM attribution.
- Ensure pull requests (PRs) created from LLM-generated changes meet quality, linting, testing, documentation, and merge requirements.

Required for every created or modified source file
1. License header
   - Include the project license header (usually Apache-2.0) at the top of every source file.
   - Add an SPDX identifier where applicable: `SPDX-License-Identifier: Apache-2.0`.
   - Ensure a matching `LICENSE` file exists at the repository root.

2. LLM attribution
   - Add a short statement in each generated/modified file indicating it was produced or modified with the assistance of an LLM.
   - Example (file header):
     `This file was created or modified with the assistance of an AI (Large Language Model). Please review for correctness and security.`

Recommended file header template (adjust comment style per language)
```
/*
 * Copyright <YEAR> <OWNER>
 * SPDX-License-Identifier: Apache-2.0
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * This file was created or modified with the assistance of an AI (Large Language Model).
 * Review required for correctness, security, and licensing.
 */
```

Pre-PR requirements (must be completed before opening a PR)
- Linting: Run the repository's linters and fix all issues. Confirm the lint score is 10/10 (or the project's maximum scoring metric). Provide the exact command used.
- Tests: Run unit/integration tests and ensure they pass locally. Add tests for behavior changes introduced by the PR.
- Static analysis/type checks: Run tools like mypy, go vet, staticcheck, etc., and fix issues.
- Formatting: Run configured formatters (black, gofmt, prettier, etc.).
- Pre-commit hooks: Run and fix any pre-commit failures.
- CI: Ensure the branch builds and passes CI checks (run locally where possible or rely on CI after push).
- Merge conflicts: Rebase or merge the latest base branch and confirm there are no conflicts.
- Changelog/versioning: If user-facing behavior changed, update CHANGELOG.md or follow repository release practices.

Documentation requirements
- If the PR introduces or changes functionality, update relevant documentation:
  - README.md (usage, quick start)
  - docs/ pages or site content describing implementation/behavior
  - API docs, migration guides, and "future work" notes as applicable
- Add or update examples and quick-start snippets for changed behavior.

PR content and quality expectations
- Title: concise, descriptive, imperative mood.
- Description: link to related issue(s) if any; short summary; rationale; tests added; docs updated; backward-compatibility or migration notes.
- LLM involvement: include an explicit line listing files created/modified by the LLM and a short note about human review performed.
- Validation commands: list exact commands used to validate lint, tests, and formatting.
- Attach logs or screenshots if the change affects UI or developer UX.
- Commit messages: follow the repository's convention (e.g., Conventional Commits). Keep commits atomic and well-scoped.

PR checklist (include in PR description or use as template)
- [ ] License header added to new files and top-level LICENSE present
- [ ] LLM attribution added to modified/generated files
- [ ] Linting completed — score: 10/10 (command: _______)
- [ ] Tests pass locally (command: _______)
- [ ] Static analysis/type checks pass (command: _______)
- [ ] Formatting applied (command: _______)
- [ ] Pre-commit hooks pass
- [ ] CI build passes
- [ ] No merge conflicts with base branch
- [ ] Changelog/versioning updated (if applicable)
- [ ] Documentation updated (README, docs/, API docs)
- [ ] Examples/quick-start updated (if applicable)
- [ ] Commit messages follow repository conventions
- [ ] PR description includes: issue link, summary, rationale, tests, docs, migration notes
- [ ] PR description explicitly lists LLM-modified files and human review notes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/icecake0141) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-17 -->
