# Core Requirements

- The end goal is to generate an STL file. Other types can be used for intermediate steps for accuracy (and is encouraged), but the final output should be in float32 to adhere to the STL format.

## Code Quality Requirements

- Follow standard Go conventions and best practices
- Use clear, descriptive variable and function names
- Add comments to explain complex logic or non-obvious implementations
- Include GoDoc comments for all:
  - Packages
  - Functions and methods
  - Types and interfaces
  - Exported variables and constants
- Write unit tests for core functionality
- Keep functions focused and manageable (generally under 50 lines)
- Use error handling patterns consistently

---
> Source: [github/gh-skyline](https://github.com/github/gh-skyline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-31 -->
