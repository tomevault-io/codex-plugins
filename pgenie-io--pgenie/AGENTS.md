# Dependency sources

Study the contents of the `cabal.project` file. In the `source-repository-package` sections you'll see references to repositories of dependencies that are not on Hackage. Load them into memory.

# Instructions

- Avoid polluting the top namespace with definitions that could be placed in `where` blocks. Alternatively introduce helper modules if the definitions are more general. Place the helper modules in a deeper namespace of the current module, e.g. `A.B.Helper` for `A.B`.

---
> Source: [pgenie-io/pgenie](https://github.com/pgenie-io/pgenie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-21 -->
