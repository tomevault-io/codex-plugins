# Copilot Instructions

## Project Guidelines
- When modifying StreamWrapper, make the COM `stream` field nullable and only set it to null in Dispose; avoid using `Marshal.GetIUnknownForObject` or relying on the wrapper implementing IDisposable because COM interop may be disabled.

---
> Source: [Bassman2/MediaDevices](https://github.com/Bassman2/MediaDevices) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-06 -->
