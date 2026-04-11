# Copilot Guidelines for Java Development on the Redops Project

## Purpose
Provide clear, concise rules for using Copilot and contributing Java code to the Redops project (an attack-simulation tool). These guidelines help keep code readable, testable, and secure.

## Structure and conventions
- Follow standard Java naming conventions (lowercase package names, PascalCase for classes).
- Organize code by responsibility: `com.redops.library` → `config`, `pipeline`, `modules`, `tools`, etc.
- Keep classes small and focused (Single Responsibility Principle).

## Code quality
- Use clear, descriptive names for variables and methods.
- Add comments only to explain non-obvious decisions (avoid redundant comments).
- Follow the project's style rules (indentation, reasonable line length).

## Testing
- Add JUnit unit tests for any new feature or bug fix.
- Cover edge cases (null values, empty collections, I/O errors, invalid parameters).
- For asynchronous or multi-threaded code, include at least one concurrent-behavior test when appropriate.

## Documentation
- Document public classes and methods with concise Javadoc (purpose, parameters, return values, thrown exceptions).
- Update README and the `docs/wiki/` documentation for any notable API or configuration changes.

## Security and ethics
- Never commit real sensitive data (credentials, keys) to the repository.
- Validate user inputs and handle errors safely to avoid leaking sensitive information.
- For any attack-simulation feature, add a clear note about ethical usage and legal prerequisites in the docs.

## Versioning & workflow
- Work on branches derived from `main` and open pull requests for changes.
- Make atomic, descriptive commits (see `git-commit-instructions.md`).
- Include automated tests in the PR when possible.

## Using Copilot
- Provide a short, precise context before asking for generated code (e.g., method signature, invariants expected).
- Manually review all generated code for security, performance, readability, and tests.
- Prefer generating small pieces (short methods, unit tests) rather than entire features at once.

## Useful templates
- Example prompts to keep near the code when using Copilot:
  - "Generate a Java method that validates a `PipelineInfo` object: return `true` if all required fields are non-null and valid. Add a JUnit test for valid and invalid cases."
  - "Create an immutable utility class for parsing YAML configuration and include Javadoc."

## Quick PR checklist
- [ ] Code compiles and tests pass locally
- [ ] New features are covered by tests
- [ ] Documentation updated if needed
- [ ] No secrets or sensitive data committed
- [ ] Commit message follows `git-commit-instructions.md`

---
Short version: use Copilot as an accelerator, not as the single source — always review, test, and secure the generated code.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Flotss)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Flotss)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
