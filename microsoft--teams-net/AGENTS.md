# Repository guidance

Release validation may run on Windows as well as Linux. Keep filesystem code and tests cross-platform: use .NET path APIs instead of hard-coded separators, specify UTF-8 when text APIs would otherwise use platform-default encodings, and avoid locale-dependent or Unix-only filesystem assumptions.

---
> Source: [microsoft/teams.net](https://github.com/microsoft/teams.net) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-09 -->
