# Shard AI Engineering Notes
- Target app is a desktop text workbench built with iced::application using the Elm update loop; keep state mutations inside Shard::update and return Task<Message>.
- Main entry lives in [src/main.rs](src/main.rs); expand the Shard state instead of adding global statics.
- Follow the UX plan in [doc/requirements.md](doc/requirements.md): top toolbar for regex controls, primary text_editor center-left, pinned snippet list on the right.
- Consult [doc/best-practices.md](doc/best-practices.md) for iced text_editor usage, highlighting hooks, and async Task patterns before wiring input handling.
- Prefer iced::widget::text_editor with .on_action to capture edits; keep the editor content in a dedicated text_editor::Content field on Shard.
- For syntax highlighting, pipe editor content through iced::highlighter; pick themes that ship with iced master to avoid custom asset loading.
- Regex batch processing should use the regex crate already listed in Cargo.toml; surface parse errors inline (no panics).
- Clipboard listening must leverage arboard in an iced::Task spawned from update when the toggle is enabled; remember to gate Windows-specific code with cfg.
- Stick with iced::Theme::Dark as a base; expose per-widget styles via theme() overrides rather than inline colors for consistency.
- Keep messages grouped by feature domain (EditorMessage, RegexMessage, SnippetMessage) and route through Shard::update using enums.
- Snippet storage belongs in a Vec<Snippet> on Shard; each entry should include title/preview plus the full text for reload/copy actions.
- Async work (regex replace, clipboard read) should return Task::perform futures built atop tokio since the runtime feature is enabled.
- Avoid Command; iced master requires Task::perform or Task::none when returning from update.
- Use immutable builders for widgets (column!, row!, scrollable) and compose them in Shard::view; keep layout constants near the view code.
- Derive Default for sub-state structs to simplify reset flows (e.g., RegexPanelState, SnippetState).
- Build locally with cargo build; watch for iced git updates that may require cargo update --aggressive if API drift occurs.
- Run cargo fmt --all and cargo clippy -- -D warnings before submitting changes; iced master evolves quickly so fix new lints immediately.
- Execute cargo test for any logic you factor into pure modules (e.g., regex utilities) even though GUI code remains untested.
- When adding modules, expose them via mod declarations in src/main.rs until the project grows enough to warrant src/lib.rs.
- Respect the Task-based lifecycle: fire-and-forget background jobs should use Task::perform with Message variants to capture completion.
- Keep enums exhaustive; use _ => unreachable! sparingly and only behind debug_assert! to avoid crashing production builds.
- Reuse iced::widget::Scrollable for the pinned list; cap snippet previews (e.g., first 200 chars) to maintain layout stability.
- Surface status text via iced::widget::text at the bottom of the main column (mode indicators, regex errors) per the requirement spec.
- If introducing file IO, gate it behind tokio::fs APIs to align with the async ecosystem already enabled.
- Document new workflows inside doc/best-practices.md so future agents inherit the knowledge base.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hewel) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
