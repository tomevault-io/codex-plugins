
# Codable Boundary

- Do not return `Codable` transport payload types (`*JSON`, DTOs, wire models) from service or repository APIs.
- Decode transport payloads at the boundary, then map to app-facing contracts before returning.
- Service and repository protocols should expose domain models (`Models/`) or explicit feature-layer data contracts only.
- Keep transport model placement under infrastructure boundaries (for example `Services/API/`), not `Models/`.

---
> Source: [Homebrew/BrewUI](https://github.com/Homebrew/BrewUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-16 -->
