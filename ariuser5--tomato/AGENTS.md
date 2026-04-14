# Copilot Instructions for tomato

## Project Purpose
- This repository orchestrates accounting and monthly-report workflows with PowerShell.
- The entrypoint is `Start-Main.ps1`, which discovers and runs automations from JSON metadata.
- Shared/base workflow building blocks live under `base/` (`base/helpers`, `base/utils`, `base/tomatoflow`).
- Configuration entrypoints live at the repository root under `conf/`.
- The repository root is intentionally kept open for extender/custom overlays that should not be part of the base image.

## Automation Conventions
- Keep automation commands compatible with `base/helpers/AutomationConfig.psm1` expectations (`alias` + `command`, with optional `args` and `cwd`).
- Assume `Start-Main.ps1` sets these env vars before running automations:
  - `TOMATO_ROOT`
  - `BASE_DIR`
  - `UTILS_ROOT`
- For utility refactors, prefer small composable scripts that orchestrate reusable module functions.
- Keep workflow-agnostic utilities under `base/utils`.
- Keep Tomatoflow-specific building-block scripts under `base/tomatoflow/automations/scripts/*.ps1` and reusable helpers in:
  - `base/tomatoflow/automations/scripts/modules/*.psm1`
- Keep generic reusable helpers in:
  - `base/utils/common/*.psm1`
- Reuse shared command wrappers (`Assert-RcloneAvailable`, `Invoke-Rclone`, `Test-ExecutableAvailable`) instead of ad-hoc command invocations.

## PowerShell Style
- Use `[CmdletBinding()]` and a `param()` block for new scripts.
- Keep `$ErrorActionPreference = 'Stop'` in scripts that orchestrate workflows.
- Fail fast with clear messages when required files, paths, or config values are missing.
- Preserve existing script behavior and avoid changing interactive UX unless explicitly requested.
- Use `Write-Host` for human-readable progress/status logs.
- Use `Write-Output` only for machine-consumable final results, ideally as one structured object (for example `New-ToolResult`).

## JSON and Paths
- Treat JSON config as user-editable; validate required fields before execution.
- Support both relative and absolute paths where possible.
- Do not hardcode machine-specific absolute paths in committed scripts.

## Changes and Safety
- Make minimal, targeted edits that preserve current public aliases and commands.
- Avoid introducing new dependencies if PowerShell built-ins can solve the task.
- Update README or samples when new automation parameters or flows are introduced.
- When adding/changing functionality under `base/`, update both:
  - `base/docs/CATALOG.md` (capability catalog)
  - `base/README.md` when package scope/boundaries or high-level guidance changed.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ariuser5) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
