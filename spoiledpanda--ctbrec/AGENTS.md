

# Copilot Instructions for ctbrec (JavaFX snapshot)

These instructions help AI coding agents work productively in this codebase snapshot.

## Big Picture
- This is a compiled Java/JavaFX desktop app. Only `.class` files and JavaFX CSS are present—**no Java source or build configs**.
- UI is under `ui/`, with per-site modules in `ui/sites/<site>/` (e.g., `chaturbate`, `camsoda`).
- Event/data flow: runtime events (e.g., `PlayerStartedEvent.class`) → DTOs (`ui/io/json/dto/`) → mappers (`ui/io/json/mapper/`).
- Core classes: `CamrecApplication`, `Launcher`, `Player`, `JavaFxModel`, `JavaFxRecording`, `RecordingDownload.class`.

## Safe Edit Scope
- **Allowed:** JavaFX CSS in `ui/controls/**`, `ui/settings/**`, `ui/tabs/**` (e.g., `Popover.css`).
- **Allowed:** Documentation (README, this file, future docs/assets).
- **Forbidden:** Do **not** modify, remove, or add `.class` files (compiled artifacts).

## Project Patterns & Conventions
- Per-site UI modules: `<Site>SiteUi.class`, `<Site>TabProvider.class`, `<Site>ConfigUi.class`, `<Site>UpdateService.class` in `ui/sites/<site>/`.
- Event → DTO → Mapper: e.g., `PlayerStartedEvent.class` → `PlayerStartedEventDto.class` → `PlayerStartedEventMapper.class`.
- JavaFX CSS: Use component-scoped classes (e.g., `.popover`, `.popover-frame`), `-fx-background-*`, `-fx-effect`, `-fx-shape`.

## CSS Styling Guidance
- Keep selectors and `-fx-*` tokens intact. Adjust values (size, color, effect) but do not change selector structure.
- Example: update `.popover-title` or `.popover .button` in `ui/controls/Popover.css` for typography tweaks.
- Do **not** rename selectors or remove class names. Tune effects (e.g., `dropshadow`) rather than replacing them.
- Test across themes if possible; keep color usage tied to variables like `-fx-base`.

## Developer Workflows (Snapshot Limits)
- **No build/test configs** (Maven/Gradle, unit tests) are present—cannot build or run Java from this folder alone.
- For Java code changes, request the source repo, build tool, and JDK/JavaFX version.

## Build/Run (when source is available)
- JDK 21 and JavaFX 21.x required (classfile major 65 detected).
- See `docs/BUILD_TEMPLATES.md` for Maven/Gradle templates.
- Discover JavaFX modules:
  - `jdeps -q -s -multi-release 21 -cp . ui\CamrecApplication.class`
- Example Maven run: `mvn -q clean javafx:run`
- Example Gradle run: `gradlew run`

## Do / Don’t for Agents
- **Do:** Propose CSS/UI tweaks with precise file paths/selectors.
- **Do:** Reference exact class files for behaviors (e.g., `ui/sites/chaturbate/ChaturbateUpdateService.class`).
- **Don’t:** Invent build commands or introduce Java sources without the build system.
- **Don’t:** Remove or rename compiled `.class` files.

## If Unclear
- If build/run/test instructions are needed, request the source repo, build tool, and JavaFX version.
- If you find undocumented patterns, add them here for future agents.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SpoiledPanda)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/SpoiledPanda)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
