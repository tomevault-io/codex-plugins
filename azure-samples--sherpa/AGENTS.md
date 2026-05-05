# Sherpa MCP Security Workshop — Copilot Instructions

## Repository Structure

```
sherpa/
├── camps/                       # Workshop modules
│   ├── base-camp/               # Local-only MCP fundamentals
│   ├── camp1-identity/          # OAuth, Managed Identity, Key Vault
│   ├── camp2-gateway/           # APIM gateway, Content Safety, API Center
│   ├── camp3-io-security/       # Input validation, output sanitization, PII
│   └── camp4-monitoring/        # Logging, dashboards, alerts, incident response
├── docs/                        # MkDocs documentation site
│   └── camps/                   # Workshop guides (mirrors camps/ structure)
├── infra/                       # Shared infrastructure docs
└── mkdocs.yml                   # MkDocs configuration
```

## Camp Conventions

Each Azure camp (1–4) follows this layout:

| Path | Purpose |
|------|---------|
| `azure.yaml` | azd project definition with `hooks:` for `preprovision` and `postprovision` |
| `infra/` | Bicep modules for Azure resources |
| `scripts/` | Waypoint scripts following **exploit → fix → validate** pattern |
| `scripts/hooks/` | azd lifecycle hooks (preprovision, postprovision) |
| `tests/` | Test and example scripts |
| `servers/` | MCP server and API source code (Python, FastMCP) |

### Script Naming

- **Numbered waypoints:** `{section}.{waypoint}-{action}.sh` — e.g., `1.1-deploy.sh`, `2.1-validate.sh`
- **Functional names:** `register-entra-app.sh`, `get-mcp-token.sh` (Camp 1 uses this style)
- **Subdirectories:** Camp 4 uses `scripts/section{N}/` subdirectories
- **Utility libraries:** Some camps have shared functions (e.g., `common.sh` sourced by other scripts)

### azure.yaml Hooks

When a camp has both `.sh` and `.ps1` hooks, the `azure.yaml` uses platform-specific sections:

```yaml
hooks:
  preprovision:
    posix:
      shell: sh
      run: ./scripts/hooks/preprovision.sh
      continueOnError: false
    windows:
      shell: pwsh
      run: ./scripts/hooks/preprovision.ps1
      continueOnError: false
```

## Windows Support

Every `.sh` script should have a corresponding `.ps1` PowerShell equivalent. Camp 2 is the reference implementation with full parity. Use the `/windows-scripts` skill for conversion and `/windows-docs` skill for documentation updates.

## Documentation

- Built with **MkDocs Material** (`mkdocs.yml` at repo root)
- Tab syntax for OS-specific commands uses `=== "Bash"` / `=== "PowerShell"` content tabs
- Indentation matters — tabs inside admonitions (`???`, `!!!`, `???+`) require extra 4-space indent
- Cross-platform commands (e.g., `azd provision`, `az account show`) do NOT need tabs

## Code Guidelines

- **Python:** 3.11+, type hints, `uv` for package management
- **Bash scripts:** `set -e` (or `set -euo pipefail`), clear progress output with section headers
- **PowerShell scripts:** `$ErrorActionPreference = 'Stop'`, use `curl.exe` (not the PowerShell alias), use `ConvertFrom-Json` / `ConvertTo-Json` instead of `jq`
- **Bicep:** Consistent naming with `abbreviations.json`, security-focused comments

---
> Source: [Azure-Samples/sherpa](https://github.com/Azure-Samples/sherpa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-04 -->
