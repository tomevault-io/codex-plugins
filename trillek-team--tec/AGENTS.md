# Copilot Instructions for Trillek Engine C

## 🚨 CRITICAL: Read AGENTS.md First 🚨

**Before doing ANY work, read `AGENTS.md` completely.** It contains:
- Mandatory environment setup steps (vcpkg, CMake)
- Execution protocol and workflow
- Documentation policy (what NOT to create)
- Build verification requirements
- Coding standards
- All project constraints and guidelines

**AGENTS.md is the single source of truth for all project rules.**

## Quick Reference (Full details in AGENTS.md)

**⚠️ PREREQUISITES FIRST**: Before building, you MUST install:
1. **System deps** (Linux): `sudo apt-get install -y build-essential pkg-config tar curl zip unzip gdb libgl1-mesa-dev xorg-dev libglu1-mesa-dev libxinerama-dev libxcursor-dev p7zip-full`
2. **vcpkg**: Clone, bootstrap, set `VCPKG_ROOT` environment variable
3. **CMake**: 3.20 or higher
4. **Ninja**: Build tool (optional but recommended)

**Build Commands**:
- **Configure**: `cmake --preset=ninja-multi-vcpkg`
- **Build Debug**: `cmake --build --preset=ninja-multi-vcpkg-debug`
- **Build Release**: `cmake --build --preset=ninja-multi-vcpkg-release`
- **Build Release with Debug Info**: `cmake --build --preset=ninja-multi-vcpkg-release-dbginfo`
- **Tests**: Enable with `-DBUILD_TESTS=ON`, then run `ctest --preset=ninja-multi-vcpkg-debug` or `ninja-multi-vcpkg-release`
- **Documentation**: `cmake --build --preset=ninja-multi-docs` (requires doxygen, doxybook2, mkdocs)

**Presets**: Use `ninja-multi-vcpkg` config preset with appropriate build preset. Ninja Multi-Config generator is preferred.

**Project Options**:
- `BUILD_CLIENT` (default ON) - Build the client
- `BUILD_SERVER` (default ON) - Build the server
- `BUILD_TESTS` (default OFF) - Build tests
- `BUILD_DOCS` (default OFF) - Build documentation

**Note**: 
- `VCPKG_ROOT` must be set as an environment variable
- Linux requires `gcc10/g++10` or `clang-13` minimum
- Project uses traditional headers (`.hpp`/`.cpp`), not C++20 modules

**Tech Stack**: C++20, CMake 3.20+, vcpkg, protobuf (networking), Bullet3 (physics), ImGui (debug UI), GLFW3 (windowing), OpenGL (rendering), OpenAL (audio), Lua + sol3 (scripting), spdlog (logging)

**Rule of Thumb**: If you're unsure about anything, check AGENTS.md.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/trillek-team)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/trillek-team)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
