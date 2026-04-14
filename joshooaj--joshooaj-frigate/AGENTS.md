<!-- GitHub Copilot / AI agent guidance for joshooaj.frigate -->
# Copilot instructions — joshooaj.frigate

Purpose: give an AI coding agent the essential, repo-specific context to be immediately productive.

1) Big picture
- This project is a PowerShell module that wraps the Frigate API. The module lives in `joshooaj.frigate/` and is packaged as a PowerShell module (`.psm1` / `.psd1`).
- Public command scripts are under `joshooaj.frigate/Public/`; internal helpers are under `joshooaj.frigate/Private/`.
- The module entrypoint `joshooaj.frigate.psm1` dot-sources every script in `Public` and `Private`, then exports public functions by basename. See `joshooaj.frigate/joshooaj.frigate.psm1`.

2) Build, test, and common workflows
- Primary build/test driver: `build.ps1` (PSake wrapper). Typical commands:
  - Run default (build + tests): `./build.ps1` or use VS Code Task "Build"/"Test" defined in workspace tasks.
  - Run tests only: `./build.ps1 -Task Test` or use the "Test" task.
  - Bootstrap dependencies: `./build.ps1 -Bootstrap` (installs PSDepend and invokes `requirements.psd1`).
- `psakeFile.ps1` configures PSBPreference values and runs `dotnet tool restore`. The test step uses Pester; outputs land under `out/testResults.xml`.
- Versioning: `nbgv` (dotnet tool) is used to set `ModuleVersion` during build (see `psakeFile.ps1` line that calls `dotnet nbgv get-version`). Ensure `dotnet` and `nbgv` are available.

3) Packaging & output
- The build emits a module under `Output/joshooaj.frigate/<version>/` (psd1/psm1 + localized help in `en-US/`). Tests and packaging expect the output layout.
- Manifest: `joshooaj.frigate/joshooaj.frigate.psd1` is the module manifest template; the build updates `ModuleVersion`.

4) Tests & expectations
- Tests live in `tests/` and heavily rely on the built module in `Output/...`:
  - `tests/Help.tests.ps1` imports the built module manifest and asserts help content (examples, descriptions, related links).
  - `tests/Manifest.tests.ps1` validates the manifest and compares changelog version.
- Environment variables used by tests: `BHProjectName`, `BHBuildOutput`, `BHProjectPath`. The PSake build sets up the build environment so tests run correctly under the `build.ps1` workflow.

5) Conventions & patterns an agent must follow
- Export pattern: do not add exports manually to `psd1` — runtime `psm1` dot-sources scripts and exports public functions by basename (`Export-ModuleMember -Function $public.Basename`). When adding commands, add the script to `Public/` and include proper `Get-Help` comment-based help.
- Help & docs: command help pages live in `docs/en-US/*.md` and are packaged to `Output/.../en-US/`. `Help.tests.ps1` expects complete help (synopsis, description, examples).
- Keep `Public/` functions small and testable; prefer helper functions in `Private/` for shared logic.

6) Integration points & external deps
- External/tooling to have: `dotnet` (for `nbgv`), `nbgv` dotnet global tool, PowerShell Gallery modules via `requirements.psd1` (installed by `./build.ps1 -Bootstrap`).
- Network: tests call external URIs from help links (see `tests/Help.tests.ps1`) — CI may need network access.

7) Where to look first (quick map)
- build driver: [build.ps1](build.ps1#L1-L80)
- psake: [psakeFile.ps1](psakeFile.ps1#L1-L120)
- module entry: [joshooaj.frigate/joshooaj.frigate.psm1](joshooaj.frigate/joshooaj.frigate.psm1#L1-L60)
- manifest: [joshooaj.frigate/joshooaj.frigate.psd1](joshooaj.frigate/joshooaj.frigate.psd1#L1-L80)
- public commands: `joshooaj.frigate/Public/` (each `.ps1` is a command)
- tests: `tests/` (Pester-based; see `Help.tests.ps1` and `Manifest.tests.ps1`)
- docs/help: `docs/en-US/`

8) Example tasks for an AI agent
- To run tests locally in correct context: `./build.ps1 -Task Test -Bootstrap`.
- To update a command's help, edit `joshooaj.frigate/Public/<Command>.ps1` and its corresponding `docs/en-US/<Command>.md`, then run `./build.ps1 -Task Test`.

If anything above is unclear or you need the file to emphasise additional patterns (CI, release process, or preferred test edits), tell me which area to expand.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joshooaj) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
