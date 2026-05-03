git git# GitHub Copilot Instructions for HOBL

## PowerShell Coding Standards

### Error Message Formatting
When logging or displaying error messages in PowerShell scripts:
- Always use the format: `" ERROR - "` (note the spacing)
- Leading space before ERROR
- Single space after ERROR
- Dash character `-`
- Single space after the dash
- Example: `" ERROR - Last command failed."`
- Example: `Write-Host " ERROR - Unsupported architecture" -ForegroundColor Red`
- Example: `if ($msg -Match " ERROR - ") { ... }`

### Defensive Programming and Path Handling
- **Never assume default installation paths** unless explicitly instructed to do so
- Always use discovery mechanisms to find installed software:
  - For Visual Studio: Use `vswhere.exe` to query actual installation paths
  - For other tools: Use registry queries, environment variables, or PATH lookups
- Verify paths exist before using them in operations
- Log the actual paths found for debugging and transparency
- Support non-default installation locations (e.g., custom drives, custom folders)
- Example: Use `getVSVersion` function to find VS, then use `$vsInfo.Path` instead of hardcoded `$vsInstallPath`

### Drive-Relative Paths (No Hardcoded C: Drive)
- **Never hardcode `c:\` or any drive letter** in Windows scenario scripts — the scripts may run from any drive
- Always derive the drive letter from the script's own location using `$PSScriptRoot`:
  ```powershell
  $scriptDrive = Split-Path -Qualifier $PSScriptRoot
  ```
- Use `$scriptDrive` to construct all HOBL working paths (`hobl_data`, `hobl_bin`, repo clones, temp dirs, etc.)
- For `param()` block defaults: use an empty string and set the real default after `$scriptDrive` is computed:
  ```powershell
  param(
      [string]$logFile = ""
  )
  $scriptDrive = Split-Path -Qualifier $PSScriptRoot
  if (-not $logFile) { $logFile = "$scriptDrive\hobl_data\scenario_prep.log" }
  ```
- **System paths are the exception** — paths under `$env:ProgramFiles`, `$env:USERPROFILE`, `$env:TEMP`, etc. are fine since they resolve to the correct OS drive automatically
- When reviewing existing scripts, replace any bare `c:\hobl_data`, `c:\hobl_bin`, `c:\opencv`, etc. with `$scriptDrive\...`

### Visual Studio Integration
- Use `vswhere.exe` to locate Visual Studio installations (handles all editions and custom paths)
- The `getVSVersion` function returns actual installation path - always use it
- Never hardcode paths like `"${env:ProgramFiles(x86)}\Microsoft Visual Studio\2022\BuildTools"`
- Verification functions should query for actual paths, not assume defaults
- Example pattern:
  ```powershell
  $vsInfo = getVSVersion -product $vsProduct
  $actualVSPath = $vsInfo.Path
  $vsDevCmd = Join-Path $actualVSPath "Common7\Tools\VsDevCmd.bat"
  ```

### pyenv-win and Python Path Resolution
- HOBL uses `pyenv-win` for Python version management across scenarios — do NOT replace it with simple `winget install` of Python
- **Never use `Get-Command python` to resolve the Python executable path** — it returns the pyenv batch file shim, not the actual `python.exe`
- Always use `pyenv which python` to get the real Python executable path
- Example:
  ```powershell
  $pythonExeRaw = pyenv which python 2>$null
  if ($pythonExeRaw) {
      $pythonExe = $pythonExeRaw.Trim()
      if (Test-Path $pythonExe) {
          "Using Python: $pythonExe" | log
      }
  }
  ```
- When passing Python paths to tools like CMake (`-DPython3_EXECUTABLE`), the path must point to the actual `.exe`, not a shim/batch file

### Execution Policy for PowerShell Archive Module
- Scripts that use `pyenv install` (which internally calls `Expand-Archive`) must set the execution policy at the top of the script
- Without this, `Microsoft.PowerShell.Archive` module fails to load on fresh Windows installs
- Always include this block early in prep scripts that use pyenv:
  ```powershell
  $executionPolicy = Get-ExecutionPolicy -Scope Process
  if ($executionPolicy -eq "Restricted" -or $executionPolicy -eq "Undefined") {
      Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope Process -Force -ErrorAction Stop
  }
  ```

### General Guidelines
- Use consistent error handling patterns across all scenario scripts
- Maintain architecture-aware code (x64 vs ARM64) where applicable
- Follow existing function patterns for check(), checkCmd(), log(), etc.
- Verify all prerequisites exist before proceeding with operations
- Provide clear error messages indicating what's missing and how to fix it

### Developer Scenario Timing and Metrics
- Every run script must report `scenario_runtime` (in seconds) via a metrics summary banner and a CSV results file
- **Timing measurements must be consistent between macOS and Windows** for the same scenario — the same phases must be timed, and `scenario_runtime` must be computed the same way on both platforms
- Before adding or changing timing in a scenario, check the other platform's script to ensure they stay aligned
- macOS scripts may capture additional detail (user, sys, cputime via `/usr/bin/time -p`) but `scenario_runtime` must always use wall-clock time on both platforms
- **CSV key names must match between platforms** where the same value is reported on both. Use the canonical key names below.

#### Canonical CSV Key Names
The following keys are shared across platforms. Where a key appears on both macOS and Windows, it must use the same name:

**Build/compile scenarios** (shared keys):
- `scenario_runtime` — wall-clock time for the full timed portion (seconds)
- `build_time` — wall-clock time for the build phase (seconds)
- `test_time` — wall-clock time for the test phase, if applicable (seconds)
- `architecture` — CPU architecture, where reported

macOS-only additional keys (from `/usr/bin/time -p`):
- `build_user`, `build_sys`, `build_cputime`
- `test_user`, `test_sys`, `test_cputime`

**AI/inference scenarios** (shared keys):
- `scenario_runtime` — total inference/generation time (seconds)
- `time_to_first_token_ms` — time to first token in milliseconds
- `time_to_first_token_s` — time to first token in seconds
- `tokens_per_second` — token generation rate
- `total_tokens_generated` — number of tokens generated
- `total_generation_time_s` — total generation time in seconds
- `ai_model` — model name (not `model`)
- `ai_device` — device type (not `device`)
- `architecture` — CPU architecture, where reported

### Prep Version Bumping
- Each developer scenario has a `prep_version` string in its Python file (e.g., `prep_version = "2"`) that controls whether the prep script re-runs on the DUT
- **Whenever prep or run PowerShell/shell scripts are modified in a PR**, increment `prep_version` by 1 in the corresponding scenario's Python file (both Windows and macOS)
- This only needs to happen **once per PR** — not per commit
- Always increment for both platforms if both were changed, or just the affected platform if only one was modified

### Teardown Script Safety Rules
- Teardown scripts **must only clean up artifacts that the run script will recreate**
- Teardown scripts **must never remove content installed during prep** (e.g., `node_modules`, Maven caches, cloned repos, conda/micromamba environments, installed packages)
- **Never use `git clean -xfd`** in teardown — it removes all untracked/ignored files including prep-installed dependencies
- Instead, use targeted removal of specific build output directories (e.g., `out/`, `.build/`, `target/`)
- Before adding cleanup to a teardown script, verify: "Will the run script recreate this?" If not, don't remove it
- The prep/run/teardown lifecycle: prep installs dependencies (runs once), run builds/tests (runs many times), teardown cleans build outputs between runs (must not undo prep)

## Developer Scenarios
When the user refers to "developer scenarios", they mean the following 11 scenarios. Each has both macOS and Windows variants with prep and run scripts. The folder structure is as follows:

| Scenario | macOS folder | Windows folder |
|---|---|---|
| Fast API | `scenarios/MacOS/mac_fast_api/` | `scenarios/windows/fast_api/` |
| Foundry Local | `scenarios/MacOS/mac_foundrylocal/` | `scenarios/windows/foundrylocal/` |
| LLVM | `scenarios/MacOS/mac_llvm/` | `scenarios/windows/llvm/` |
| MLPerf | `scenarios/MacOS/mac_mlperf/` | `scenarios/windows/mlperf/` |
| .NET Aspire | `scenarios/MacOS/mac_net_aspire/` | `scenarios/windows/net_aspire/` |
| Node.js | `scenarios/MacOS/mac_nodejs/` | `scenarios/windows/nodejs/` |
| Ollama | `scenarios/MacOS/mac_ollama/` | `scenarios/windows/ollama/` |
| OpenCV Build | `scenarios/MacOS/mac_opencv_build/` | `scenarios/windows/opencv_build/` |
| PyTorch Inference | `scenarios/MacOS/mac_pytorch_inf/` | `scenarios/windows/pytorch_inf/` |
| Spring Pet Clinic | `scenarios/MacOS/mac_spring_petclinic/` | `scenarios/windows/spring_petclinic/` |
| VS Code | `scenarios/MacOS/mac_vscode/` | `scenarios/windows/vscode/` |

When asked to make a change across "all developer scenarios", apply it to all 11 (both platforms unless a specific platform is specified).

---
> Source: [microsoft/HOBL](https://github.com/microsoft/HOBL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-03 -->
