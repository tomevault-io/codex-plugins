
# Build Mode Rules

- After making code changes that should be verified with a build, configure with `cmake --preset vs2022-x64 -DRO_ENABLE_DEV_DEPLOY=ON`.
- Build with `cmake --build --preset build-release-x64`.
- Treat this x64 Release + `RO_ENABLE_DEV_DEPLOY=ON` flow as the default build mode for this workspace unless the user explicitly asks for a different configuration.

---
> Source: [Sziadan/open-midgard](https://github.com/Sziadan/open-midgard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-03 -->
