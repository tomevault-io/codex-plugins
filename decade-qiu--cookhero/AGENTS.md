# Global Instructions

## 0. General Code Quality Requirements
- Clarity and Simplicity: Write clean, self-documenting code with meaningful names and consistent formatting. Prefer straightforward solutions; avoid unnecessary abstraction or complexity.
- Reliability and Correctness: Ensure code is robust and predictable. Handle errors gracefully, validate inputs, and include tests where appropriate.
- Maintainability and Modularity: Structure code to be easy to understand, modify, and extend. Minimize coupling between modules and keep related functionality cohesive.
- Consistency and Standards: Follow established project conventions and language best practices consistently. Avoid introducing special cases without strong justification.

## 1. Task Decomposition & Planning
- For complex or multi-step tasks, carefully analyze and plan before proceeding.
- Break down tasks into a prioritized to-do list of executable steps.
- Process each subtask sequentially by priority, ensuring logical progression and clarity.

## 2. Version Compatibility Policy
- **No backward compatibility guarantee** — Feel free to break old formats to prioritize new features and improvements.
- Allow destructive updates, prioritize code quality and architectural soundness over maintaining legacy support.

## 3. Git Operations
- **Never automatically execute git commit** after making code changes.
- Perform git operations (e.g., commit, push) only when explicitly instructed by the user.

# Python Project Instructions

## 1. Python Environment Management
- Always activate the virtual environment in the project root before running or testing code.
- Activation command: `source .venv/bin/activate`

# Frontend Project Instructions

## 1. Frontend Build Process
- After modifying frontend code, always run the following command to compile and check for errors: `npm run build`.
- Ensure the build succeeds before proceeding with other operations to maintain code consistency and stability.

---
> Source: [Decade-qiu/CookHero](https://github.com/Decade-qiu/CookHero) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-20 -->
