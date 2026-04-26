# GitHub Copilot Instructions for this Workspace

## Project Context
This project is a save decrypter/encrypter for the Sony PlayStation 4. Build automation is handled via CMake. Security, reliability, and code quality are prioritized, with static analysis and linting tasks (Flawfinder, Cppcheck, PVS-Studio, clang-format) included in the development workflow.

## Coding Standards and Style

- **Language:** C++17. The C++17 standard is enforced in CMakeLists.txt with `CMAKE_CXX_EXTENSIONS OFF`, meaning strict ISO C++17 — no GNU or compiler-specific extensions.
- **PVS-Studio header:** Every C++ file must begin with the two-line PVS-Studio comment before any other code:
  ```c
  // This is an open source non-commercial project. Dear PVS-Studio, please check it.
  // PVS-Studio Static Code Analyzer for C, C++, C#, and Java: https://pvs-studio.com
  ```
- **C file include grouping:** In C++ source files, group #include directives at the top of the file by category, in the following order: Self (this file's header), C (standard library and POSIX), C++ (if any), Other libraries (third-party), This Project's (other headers from this repo), and Banned (headers enforcing banned APIs). Each group should be separated by a blank line and include a comment header (e.g., `// C`, `// SDK`).
  - **Always keep all group comment headers**, even if a group has no includes. This makes it immediately obvious where new includes should be added. (See `sha256.c`, `fpkg.c`, etc. for examples.)
  - For each #include, add an inline comment listing, in strict case-insensitive alphabetical order, every function, constant, macro, or type from that header that is used by the C file. If a symbol is no longer used, remove it from the comment. Only include headers that directly define the symbols used; do not rely on transitive includes, as those may change. Every symbol listed should be fully qualified ex. `std::filesystem::exists` rather than `exists`. This ensures clarity and maintainability.
  - The `banned.h` header (from `external/banned`) must be included as the last include in every C file; it enforces banned APIs and helps catch unsafe code.
  - This convention improves readability and makes dependencies explicit.
  - Example:

```c
// Self
#include "common.h"

// C
#include <stddef.h>    // size_t
#include <sys/types.h> // pid_t

// C++

// Other libraries
#include "../external/inih/ini.h" // ini_parse

// This Project's
#include "notify.h" // notify

// Banned
#include "../external/banned/banned.h"
```
- **Header include guards:** All header files must use include guards with the `_H_` suffix convention: `#ifndef FOO_H_` / `#define FOO_H_` / `#endif // FOO_H_`. Header files also list used symbols in their include comments, matching the convention for `.c` files.
- **Formatting:** clang-format is used for consistent code style. Follow the style defined in the project's `.clang-format` file. Note: `SortIncludes` is enabled but `IncludeBlocks: Preserve` ensures sorting only happens within each group (separated by blank lines), complementing the manual include grouping above.
- **Comments:** Use descriptive comments for complex logic.
- **Inline suppressions:** When suppressing static analysis warnings, always include an explanation:
  - Flawfinder: `// Flawfinder: ignore - <reason why this is safe>`
  - Cppcheck: `// cppcheck-suppress <checkName> - <reason why this is safe>`
  - PVS-Studio: `// -V<number> - <reason why this is safe>`
  Suppressions without justification will not be accepted.
- **Error Handling:** Defensive programming is encouraged. Always check return values, handle errors gracefully, and avoid undefined behavior.
- **Security:** Avoid unsafe functions and patterns. Use static analysis tools regularly and address flagged issues promptly. Never use functions blacklisted by `external/banned/banned.h` (the Microsoft SDL banned API list — e.g. `strcpy`, `strcat`, `sprintf`, `gets`, `scanf`, and similar). `banned.h` is included as the last include in every `.cc` file precisely to enforce this at compile time; a build that introduces a banned function call will fail.
- **Naming:** Use clear, descriptive names for functions, variables, and files. Prefer lowercase with underscores for C code.
- **Dependencies:** External libraries are included in the external/ directory. Avoid introducing new dependencies unless necessary and approved.
- **Warnings as errors:** All warnings from the compiler and static analysis tools are treated as errors and must be resolved before merging or release. If they are false positives the suppression must explain why it's a false positive.

## Tone and Style of Communication
- **Professional and concise:** Communication should be clear, direct, and focused on technical accuracy.
- **Collaborative:** Encourage constructive feedback and knowledge sharing. Document decisions and rationale for changes.
- **Respectful:** Maintain a respectful tone in all discussions, code reviews, and documentation.
- **Documentation:** Provide thorough documentation for new features, architectural changes, and complex code. Use `README.md` and inline comments as appropriate.
- **Issue Reporting:** When reporting bugs or issues, include detailed reproduction steps, logs, and environment information.

## Specific No-Go Areas
- **No piracy or illegal content:** Do not include, reference, or facilitate piracy, or illegal content.
- **No personal data:** Do not store or process personal user data. Avoid code that collects or transmits sensitive information.
- **No unsafe code:** Avoid code that intentionally introduces vulnerabilities, backdoors, or exploits outside the intended scope.
- **No platform abuse:** Do not target or abuse PlayStation Network or Sony services. The project is strictly for homebrew and educational purposes.
- **No unapproved dependencies:** Do not add external libraries or tools without prior review and approval.

## Additional Refrences

- Full PKG implementation (Uses PFS) can be found here https://github.com/OpenOrbis/LibOrbisPkg while it may not support save data directly it's likely a good reference to have available. There's another partial refrence here https://www.psdevwiki.com/ps4/PFS

---
> Source: [Al-Azif/save-tool](https://github.com/Al-Azif/save-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-26 -->
