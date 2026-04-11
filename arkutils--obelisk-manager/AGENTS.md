# AI agent guide: obelisk-import

Purpose: a Typer-based CLI to import/manage Obelisk data files and keep a per-folder `_manifest.json`. Requires Python 3.12+, Git CLI, and uses Rich for UX. Manage everything with uv; aim to run via `uvx` or `uv tool run`.

### Primary purpose - Manifest creation
- The per-folder `_manifest.json` lists files with the minimal metadata needed for change detection and compatibility. JSON files typically contribute version/format/mod fields; binary files contribute a content hash.
- File scanner is non-recursive and skips hidden/underscored names and directories; only processes extensions in `scanner.registered_types`. Default registered extensions: `json`, `jsonc`, `png`, `jpg`, `jpeg`.
- JSON writing uses a consistent style via `src/obelisk/json_utils.py` when saving manifests and similar files:
  - Tab-indented JSON with LF newlines.
  - A small prettification step that joins certain small inline dicts onto a single line to keep manifests compact.

## Architecture
- Entry: `src/obelisk/main.py` defines the root Typer app and mounts sub-apps from `src/obelisk/commands/*`.
- Shared CLI + logging: `src/obelisk/common_args.py` exposes `DRY_RUN_ARG`, `VERBOSE_ARG`, `QUIET_ARG`, `VERSION_ARG` and `initialise_app(ctx)` which prints a banner, derives log level from -v/-q counts, and configures Rich logging via `CustomRichConsole`.
- Manifests: `src/obelisk/manifest.py` defines the `ManifestEntry` data model and the helpers `parse_manifest`, `write_manifest`, and `manifest_match`. These are used to read/write and compare per-folder manifests named `_manifest.json`. The `update-manifest` command wires these together to maintain `<folder>/_manifest.json` based on the files present.
- Scanning/registry: `src/obelisk/scanner.py` builds entries by iterating files and dispatching to handlers via `registered_types: dict[str, MetadataReader]`.
- Filetype handlers:
  - JSON: `filetypes/json.py` handles `.json` and `.jsonc` files. It reads JSON objects and, when appropriate, produces a manifest entry based on keys like `version`, `format`, and `mod`.
  - Binary: `filetypes/binary.py` handles common binary formats (e.g., images) and produces manifest entries using content-derived metadata (e.g., a fast hash) suitable for change detection.
- Commands: `commands/update_manifest.py` (manifest maintenance), `commands/live_import.py` (prints intended git actions; future implementation). Ignore `commands/hello.py` (temporary).

## Develop, run, test
- Use uv for env and commands:
  - `uv run obelisk --help`
  - `uv run obelisk update-manifest <path|_manifest.json> [--dry-run] [-v|-q]`
  - `uv run obelisk live-import -r <repo> <inputs...> <DEST_PATH> [--skip-push] [--dry-run] [-v|-q]`
- Intended future distribution/run is from PyPI via uvx: `uvx obelisk-manager --help` (enabled by `[tool.uv].package = true`).
- Tool configuration is preferred to be in `pyproject.toml`.
- Tests: pytest + doctest, split into `tests/integration` and `tests/unit`. Run via `uv run pytest`.
- Lint/formatting: Use Ruff configured in `pyproject.toml`. Check/fix using `uv run ruff check --fix [<filename>]`.
- Type checking: Pyright configured in `pyproject.toml`. Check using `uv run pyright [<filename>]`. PyLance is used in-editor for strict type checking.

## Conventions and quality bar
- Typing: prefer full annotations everywhere; use `from typing import TYPE_CHECKING` with type-only imports under `if TYPE_CHECKING:` when appropriate.
- Type checking: Respect Pyright rules.
- Lint/formatting: All Ruff issues must be resolved.
- Logging: every module should define `logger = logging.getLogger(__name__)` and rely on `initialise_app` to set Rich logging.
- Comments: explain overall flows and tricky concepts; avoid restating code.
- Testability: write small, pure functions where possible; add unit tests in `tests/unit/` and integration tests in `tests/integration/`.
- Commands pattern:
  - Include `ctx: Context` first; call `console = initialise_app(ctx)` and do `print = console.print`.
  - Include shared flags: `show_version: VERSION_ARG = False`, `dry_run: DRY_RUN_ARG = False`, `verbose: VERBOSE_ARG = False`, `quiet: QUIET_ARG = False`.

## Integration notes
- Git CLI must be available for `live-import` functionality.
- Packaging: `project.scripts` exposes `obelisk-manager = "obelisk.main:app"`; `[tool.uv].package = true` supports `uvx` installs.

## After Editing
Ensure to run the following after editing files:
- Run `uv run ruff check --fix` to auto-fix linting and formatting issues.
- Run `uv run pyright` to verify type checking passes.
- Run `uv run pytest tests/unit` to ensure all unit tests pass.
- Run `uv run pytest tests/integration` to ensure all integration tests pass.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/arkutils)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/arkutils)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
