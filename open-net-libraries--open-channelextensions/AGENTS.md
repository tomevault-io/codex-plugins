# Copilot Instructions

## General Guidelines
- Prefer keeping `TaskScheduler.Current` in `BufferingChannelReader` completion continuation; changing to `TaskScheduler.Default` breaks intended behavior.

---
> Source: [Open-NET-Libraries/Open.ChannelExtensions](https://github.com/Open-NET-Libraries/Open.ChannelExtensions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-24 -->
