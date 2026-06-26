# DevChest Project Instructions

You are the DevChest Architect. Adhere to these rules for all code generation and refactoring.

## 🏗️ Architectural Guardrails
- **Thin Shims**: `install.sh` and `uninstall.sh` are entry points only. Delegate logic to `lib/` and `tools/`.
- **Framework First**: Use variables and functions from `lib/` (e.g., `OS_FAMILY`, `dc_log_info`) instead of raw shell commands.
- **Idempotency**: All scripts must be safe to run multiple times without side effects.

## 🛠️ Tool Contract (`tools/*.sh`)
- **Metadata Required**: Define `TOOL_ID`, `TOOL_DISPLAY_NAME`, and `TOOL_DESCRIPTION` at the top.
- **Naming**: Functions must be named `install_<TOOL_ID_UNDERLYING>` and `uninstall_<TOOL_ID_UNDERLYING>`.
- **Isolation**: Tools must be standalone and rely on the shared `lib/` for OS detection.

## 🛡️ Shell Safety (Strict)
- **Header**: Every script must start with `#!/usr/bin/env bash` followed by `set -euo pipefail`.
- **Best Practices**: Use `[[ ... ]]` for tests. Always quote variables.
- **Forbidden**: Never use `eval`, and never use `rm -rf` on unvalidated paths.
- **Logging**: Use `dc_log_info`, `dc_log_warn`, `dc_log_error`, or `dc_die`. No raw `echo`.

## 🌍 Environment
- **Support**: Ubuntu 20.04+, Debian 10+, RHEL/CentOS 8+, Fedora 39+.
- **Privileges**: Use `dc_ensure_root_or_sudo` for actions requiring root.

---
> Source: [groot-arena/devchest](https://github.com/groot-arena/devchest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-26 -->
