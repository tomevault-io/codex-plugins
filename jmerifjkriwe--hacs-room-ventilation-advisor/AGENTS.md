# Coding Standards

## General Guidelines
- Use Context7 to fetch up-to-date documentation for libraries and frameworks rather than relying on training data.
- Using English for all code, comments, documentation, and commit messages is mandatory.
- Write code that is easy to read and understand
- Follow the principle of least surprise; code should behave in a way that is expected
- Use descriptive names for variables, functions, and classes
- Avoid using magic numbers; use constants instead
- Follow the DRY principle (Don't Repeat Yourself)
- Keep functions small and focused on a single task.
- Use comments to explain why something is done, not how
- Use version control (Git) for all code changes

## Commit Messages
- Follow the [Conventional Commits](https://www.conventionalcommits.org/) specification.
- This is required for the automated release process to work correctly.
- Use the following prefixes:
  - `feat`: A new feature
  - `fix`: A bug fix
  - `docs`: Documentation only changes
  - `style`: Changes that do not affect the meaning of the code (white-space, formatting, missing semi-colons, etc)
  - `refactor`: A code change that neither fixes a bug nor adds a feature
  - `perf`: A code change that improves performance
  - `test`: Adding missing tests or correcting existing tests
  - `chore`: Changes to the build process or auxiliary tools and libraries such as documentation generation
- A scope may be provided to a commit’s type, to provide additional contextual information and is contained within parenthesis, e.g., `feat(parser): add ability to parse arrays`.
- For breaking changes, add `BREAKING CHANGE:` to the footer of the commit message.
- Use consistent indentation and formatting throughout the codebase.
- Fix all ESLint issues before committing (JavaScript/TypeScript linting & formatting).
- Fix all Ruff issues before committing (Python linting & formatting).
- Fix all Pylance issues before committing (Python type checking).
## Home Assistant Guidelines
- Follow the official [Home Assistant Developer Documentation](https://developers.home-assistant.io/) as the primary reference for development.
- Use platform-specific best practices when writing custom components (e.g., for sensors, switches, services).
- Align with the Home Assistant [architecture decisions](https://developers.home-assistant.io/docs/architecture_index/) and avoid anti-patterns that contradict their standards.
- Ensure compatibility with Home Assistant’s update cycles and deprecation policies.
- Write code that can be upstreamed or reused by the Home Assistant community.
- Always refer to the latest official Home Assistant documentation instead of relying on outdated community posts or AI suggestions.
## Naming Conventions
- Use PascalCase for component names, interfaces, and type aliases
- Use camelCase for variables, functions, and methods
- Prefix private class members with underscore (_)
- Use ALL_CAPS for constants
## Code Structure
- Organize code into modules or packages based on functionality
- Use a consistent file structure across the project
- Keep related files together (e.g., components, styles, tests)
## Error Handling
- Use try/catch blocks for async operations
- Always log errors with contextual information
- Use custom error classes for specific error types
- Avoid swallowing errors silently
- Provide user-friendly error messages
- No warnings should be ignored; address them appropriately
## Documentation
- Use docstrings for all public functions, classes, and methods
- Use Markdown for README files and documentation
- Keep documentation up-to-date with code changes
- Use comments to explain complex logic or important decisions
## Testing
- Please refer to the comprehensive testing guide here: 📄 [`tests/TESTING.md`](tests/TESTING.md)
- Follow the established patterns for config flow and options flow tests
- Write both positive and negative test cases for all functionality
- Ensure tests are independent and properly use fixtures from conftest.py
## Code Reviews
- All code changes must go through peer review
- Provide meaningful descriptions in pull requests
- Review code for readability, security, and performance
- Ask for clarification rather than making assumptions
## Internationalization (i18n)
- Keep all translation files in `/translations/` up-to-date, especially `en.json`.
- Add new keys to `en.json` and `de.json` whenever UI text is added or changed.
- Do not remove keys without verifying their usage across the codebase.
- Use consistent key naming for translations to ensure clarity and maintainability.
- **All UI texts, including option labels and dropdown values, must be sourced from the translation files and must not be hardcoded in the codebase.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jmerifjKriwe)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/jmerifjKriwe)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
