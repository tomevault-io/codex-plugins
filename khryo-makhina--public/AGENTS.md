# GitHub Copilot — Assistant Instructions for this repository

## Purpose
- Provide general, workspace-wide behavior for GitHub Copilot suggestions.
- Ensure suggestions are concise, actionable, and aligned with the repository's style and conventions.

## General guidelines
- Tone: concise and direct.
  - Do not apologize or use filler phrases or such as like "Let me ..." but be direct and clear.
- Do not introduce secrets, hardcoded credentials, or copyrighted text verbatim.
- When unsure, ask a clarifying question rather than guessing.

## When producing code changes
- Keep diffs minimal and focused. Include tests and documentation when appropriate.
- Follow the repository's existing formatting and styling conventions.

## Cross-reference
- Subfolder custom instructions are complementary to these root instructions and apply when a subfolder needs project-specific behavior or overrides.
- Always apply the root rules unless a subfolder file explicitly specifies an override for that area.

## Maintainer contact
- If a suggestion touches deployment, security, or CI, request review from the repository owner before merging.

## Additional/override guidelines
- Focus on containerization best practices: small images, minimal layers, explicit versions for base images.
- Keep `docker-compose` and Kubernetes manifests declarative and idempotent.
- For shell scripts and Dockerfiles, prefer POSIX-compatible patterns where feasible and document platform specifics (Windows vs Linux).
- When modifying `README.md` or deployment docs, include exact commands to reproduce local setups and any environment prerequisites.

## General repository rules
- Be concise: give minimal, actionable suggestions.
- Prefer small, focused edits that follow the repo style.
- Always present a 1-line preamble before automated actions or tool calls.
- When unsure about intent, environment, or committing changes: ask the author.

## Development cycle (organizes how to apply the rules)
- Planning: define scope, list tasks, and create a `manage_todo_list` entry before edits.
- Implementation: prefer small patches; use `apply_patch` for edits and `create_file` for new files.
- Testing: run tests or linters only when requested.

## Local development and user files
- Put shareable local helpers in folder `local_dev`
	- Do not create a top-level `local_dev` folder.
- Keep ephemeral artifacts or generated files for the user in folder `local_user_files`
	- Do not create a top-level `local_user_files` folder.

### GIT commit
	- Commit message style: imperative, <=50-char subject, body wrapped at 72 chars explaining the rationale.
  - Do not GIT commit automatically, unless explicitly asked. Always ask for confirmation before committing. Preferably, propose the commit message and ask for confirmation before committing.

## C# and scripting conventions
- Prioritize readability, maintainability, and existing repository style.
- Prefer small, well-named functions and clear separation of concerns.
- File-scoped namespaces: `namespace My.Project.Sub;`.
- Add XML docs (`/// <summary>`) for new or modified public/internal types and methods.
- Use braces even for one-line `if`/`return` blocks for clarity.
- Private fields: use a leading underscore (`_myField`).
- Extract private helpers for blocks >5 lines or reusable logic.
- Refer to `clean_code_general_instructions.md` for detailed style and design guidelines.
- Do not use emojis in scripts or code as they may introduce run-time failures or errors.
- Do not use non-ASCII characters in code, scripts, or file names to avoid compatibility issues across different platforms and tools.
- Match the language of the target file: e.g., C# for .cs, Python for .py, YAML for compose/k8s manifests.
- Use UTF-8 encoding (for .csv files, use UTF-8 with BOM for MS Excel compatibility) and preserve line endings consistent with existing files.
- Add or update unit/integration tests when changing behavior.

## Testing conventions
- Use xUnit for C# tests, with Shouldly for assertions.
- Mocking and stubbing: use appropriate frameworks (e.g., NSubstitute for C#) to isolate units under test and avoid external dependencies. 
	- Do not suggest Moq for mocking as the library involves the inclusion of a controversial dependency called SponsorLink in August 2023.
	- Suggest and create an interface for the dependency when the dependency is simple - implement the interface with a test-specific class.
- Tests live in `<project name>.Tests`; mirror production structure and append `Tests` to filenames.
- Test names: `SubjectUnderTestOrMethodName_StateOrCondition_ExpectedOutcomeOrBehaviour`.
- Test body: `// Given: <state or condition>`, blank line, `// When: <action>`, blank line, `// Then: <expected outcome>`.
  - Arrange the test scenario code, so that, we can see clearly the Given-When-Then blocks. Example for Given: "//Given x is 1 and y is 2". Keep the setup/given in their private methods if they tend to be bloating the test scenario readability.
- Run tests or linters only when requested.

## Tooling & security
- Planning: define scope, list tasks, and create a `manage_todo_list` entry for multi-step tasks before edits.
- Implementation: prefer small patches; use `apply_patch` for edits and `create_file` for new files.
- Provide copyable commands.

## Security
- Do not include secrets, credentials, or external network calls in patches or examples.

## Final note
- This file is the authoritative assistant guide for this repo. If anything is unclear, ask the repository maintainer before proceeding.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/khryo-makhina)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/khryo-makhina)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
