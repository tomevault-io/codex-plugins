@~/.claude/rules/cpp.md

# Building

- Use CMake with preset "clang-debug" for building and testing on Linux with Clang in debug mode.
- Use CMake with preset "clang-release" for performance testing on Linux with Clang in release mode.

# Testing

- Run the tests using `ctest --preset=clang-debug` or `ctest --preset=clang-release` depending on the build type.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/christianparpart)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/christianparpart)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
