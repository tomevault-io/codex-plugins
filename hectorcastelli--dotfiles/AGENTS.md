## Purpose

This repo is a minimal, hackable dotfiles manager implemented as a set of shell scripts and a collection of "profiles". The guidance below helps an AI coding agent make focused, correct edits that follow the project's conventions.

## Big picture (what matters)

## Developer workflows & important commands
  - sh -c "$(curl -fsSL https://raw.githubusercontent.com/HectorCastelli/dotfiles/HEAD/scripts/get.sh)"
  - `sh` and `curl` are required for the bootstrap step.

## Project-specific patterns & conventions
  - `install.sh` — apply changes for that profile (idempotent). Example template is under `profiles/_template/install.sh` which begins with `#!/usr/bin/env sh` and `set -eu`.
  - `uninstall.sh` — undo what the profile installed.
  - `prompt.sh` — (optional) request interactive inputs required by `install.sh`.
  - `home/` — a directory whose files are symlinked into $HOME; treat it as the canonical declaration of desired home files.
   - `uninstall.sh` — undo what the profile installed.
   - `prompt.sh` — (optional) request interactive inputs required by `install.sh`.
   - `home/` — a directory whose files are symlinked into $HOME; treat it as the canonical declaration of desired home files.

  Script template (required for new scripts)
   - All new scripts should follow the format used in `scripts/_template.sh` so they are consistent and safe when sourced or executed.
   - Required characteristics:
     - Use the POSIX-friendly interpreter shebang: `#!/usr/bin/env sh`.
     - Use strict flags at the top: `set -eu` (error on unset variables and exit on errors).
     - Prefer named functions for actions, e.g. `function1()` and `function2()`.
     - Implement a simple `case "${1:-}"` dispatcher so the script can be sourced without side-effects. The default case should do nothing.
     - When exiting due to an error, use distinct non-zero exit codes for different error conditions.

  Example minimal template (follow this shape exactly):

  ```shell
  #!/usr/bin/env sh
  set -eu

  function1() {
    echo "func1"
  }

  function2() {
    echo "func2"
  }

  case "${1:-}" in
  function1)
    function1
    ;;
  function2)
    function2
    ;;
  *)
    # script was sourced, so we do nothing.
    ;;
  esac
  ```

   - Rationale: this pattern makes scripts safe to source (no side effects by default), explicit about actions (named functions), and robust in CI by failing fast on errors or unset variables.

## Integration points & external dependencies

## How an agent should approach common tasks

## Quick file references (examples)

If anything in this guide is unclear or you want the agent to be more prescriptive about testing, linting, or stricter script headers, tell me which area to expand and I will iterate.

---
> Source: [HectorCastelli/dotfiles](https://github.com/HectorCastelli/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-22 -->
