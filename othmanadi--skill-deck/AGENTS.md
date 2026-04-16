This is a Tauri v2 desktop app with a Rust backend (src-tauri/) and Svelte 5 frontend (src/).

Architecture: Adapter pattern for 15+ coding agents. Registry in agents/registry.rs, universal parser in parsers/frontmatter.rs, unified Skill model in models/skill.rs.

Build: `pnpm install && pnpm tauri dev`
Test: `cd src-tauri && cargo test && pnpm check`

See AGENTS.md for complete architecture documentation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/OthmanAdi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
