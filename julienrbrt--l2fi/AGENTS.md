# GitHub Copilot Instructions

These instructions guide GitHub Copilot to generate code that follows the coding standards and security practices projects.

## General Guidelines

Whenever you generate code or documentation:

1. Use extremely simple, direct language—no unnecessary adverbs.
2. Make the code self-explanatory. Only add comments when an implicit operation must be called out.
3. Follow instructions exactly. Think critically about security: always double-check for hidden bugs or vulnerabilities.
4. Produce readable yet concise code without premature abstraction.
5. When writing Go, adhere to the latest official Go best practices (idiomatic naming, error handling, package layout, etc.).
6. Adhere to Uber Go best practices.
7. Keep suggestions minimal and focused. Avoid excessive detail or overly prescriptive guidance.
8. Only use the following technologies and limit the use of external libraries::
    - Go (Golang)
        - Go Chi (github.com/go-chi/chi)
        - GORM (github.com/go-gorm/gorm)
        - Go templates
    - SQLite
        - github.com/glebarez/sqlite
    - HTMX
        - htmx.org
    - TailwindCSS

## Go-Specific Guidelines

### Test-First Development

Follow this workflow when implementing new features or fixing bugs:

1. **Write failing tests first** - Create tests that define the expected behavior before writing implementation code
2. **Run tests to confirm they fail** - Verify the tests fail for the right reasons
3. **Write minimal code to make tests pass** - Implement only what's needed to satisfy the tests
4. **Refactor with confidence** - Improve the code while keeping tests green
5. **Add edge case tests** - Expand test coverage for error conditions and boundary cases

This approach enables better iteration on code and helps catch issues early.


### Function Structure

- Keep functions focused and single-purpose
- Prefer early returns to reduce nesting
- Validate inputs at the beginning of functions
- Use guard clauses to handle edge cases early

### Testing

- **Always write tests before implementation** - Tests should define behavior and guide development
- Follow table-driven test patterns established in the codebase
- Use `testify/require` for assertions
- Name test cases descriptively to explain what is being tested
- Include both positive and negative test cases
- Test edge cases and error conditions
- Run tests frequently during development to get rapid feedback

## Code Organization

- Analyze the project structure entirely before deciding where something should go.
- Prefer standard library solutions when possible

### Linting

- Use golangci-lint before submitting

### Documentation

- Document all exported functions, types, and constants
- Use godoc-style comments that start with the item name
- Keep documentation concise
- Only document mid code for non-obvious behavior or side effects

## Pull Request Rules

- When naming a PR or commits, always follow conventional commits <https://www.conventionalcommits.org/en/v1.0.0/#summary>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/julienrbrt) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
