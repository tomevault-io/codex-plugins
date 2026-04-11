## Operater Interactions

- When asked to fix code, first explain the problems found.
- When asked to generate tests, first explain what tests will be created and why they are necessary.
- When making multiple changes, provide a step-by-step overview of the changes before implementing them.
- When asked to refactor code, first explain the reasons for the refactoring and the expected benefits.
- Use a `CHANGELOG.md` file to track changes made by Copilot

## Security

- Always validate and sanitize user input to prevent security vulnerabilities such as SQL injection or cross-site scripting (XSS).
- Use secure coding practices to protect sensitive data, such as passwords and API keys.
- Regularly update dependencies and libraries to ensure that you are using the latest security patches.
- Use environment variables to store sensitive information, such as API keys and database credentials, instead of hardcoding them in your code.
- Always use secure authentication methods, such as OAuth or JWT, to protect user accounts and data.
- Regularly review and audit your code for security vulnerabilities and potential exploits.
- Use tools like linters and static analysis tools to identify potential security issues in your code.
- Follow best practices for error handling and logging to avoid exposing sensitive information in error messages or logs.

## Environment variables

- If a .env file exists, use it to load environment variables.
- Document any new environment variables in the README.md file, but do not include sensitive information.
- Provide example values in a `.env.sample` file to guide users on how to set up their own `.env` file. Use "REDACTED" as a placeholder for sensitive information.

## Testing

- Always write unit tests for new features and bug fixes.
- Use Jest for unit tests with Testing Library and Mock Service Worker.
- Use API testing for end-to-end tests.
- Use a consistent testing strategy, such as Test-Driven Development (TDD) or Behavior-Driven Development (BDD), to ensure that tests are written before or alongside the code.
- Use a consistent naming convention for test files and test cases to improve readability and maintainability.
- Use mocking and stubbing to isolate tests and avoid dependencies on external systems or services.

## Code Quality

- Always write clean, readable, and maintainable code.
- Use meaningful variable and function names to improve code readability.
- Follow a consistent coding style and adhere to language-specific conventions.
- Use comments to explain complex or non-obvious code, but avoid over-commenting.
- Write unit tests to ensure code correctness and maintainability.
- Regularly refactor code to improve its structure and readability.
- Document your code and APIs to make it easier for others to understand and use.
- Regularly review and update documentation to keep it accurate and up-to-date.
- Use tools like Prettier or ESLint to enforce coding standards and improve code quality.
- Use tools like SonarQube to analyze code quality and identify potential issues.
- Use tools like JSDoc to generate documentation for exported functions and classes or API endpoints.
# - Use tools like Swagger or OpenAPI to document your APIs and make them easier to use.
- Use `async/await` for asynchronous code to improve readability and maintainability.
- Use `try/catch` blocks for error handling in asynchronous code.
- Use `import` and `export` statements for module imports and exports.

## Version control

- Keep commits atomic and focused on a single changes
- Commit message: Use the subject line for a concise summary of the changes with a blank line after it, followed by a longer description of the changes made if necessary.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/WhatIfWeDigDeeper)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/WhatIfWeDigDeeper)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
