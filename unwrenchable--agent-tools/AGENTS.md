# GitHub Copilot Custom Instructions — agent-tools

## Project Overview

This repository implements a reusable AI-agent registry with capability-based access control and a multi-repo rollout tool. The key components are:

- **`agent_tools/`** — Python package: registry loader, capability checker, importer, CLI (`agentx`)
- **`agent_tools/data/agents.json`** — base agent registry (30+ role-based agents)
- **`agent_tools/data/access_profiles.json`** — three-tier permission model: `safe`, `balanced`, `power`
- **`tools/rollout_all_repos.py`** — rolls out `.agentx/` packs + Copilot config to every repo in a GitHub org
- **`.agentx/`** — per-repo merged registry used at runtime
- **`.github/agents/`** — GitHub Copilot custom agent definitions (markdown format)

## Coding Conventions

- Python 3.11+; use `from __future__ import annotations`
- Dataclasses with `slots=True` for plain data containers
- `Path` objects for all filesystem operations; never raw strings
- `json.dumps(payload, indent=2) + "\n"` for every JSON write
- Slugify IDs with the `slugify()` helper (alphanumeric + hyphens, no consecutive hyphens)
- No external runtime dependencies — stdlib only in `agent_tools/`

## Agent Registry Rules

Every agent entry in `agents.json` must have:
```json
{
  "id": "kebab-case-unique-id",
  "role": "Human-readable role title",
  "description": "One-paragraph purpose statement",
  "tags": ["tag1", "tag2"],
  "capabilities": ["capability-slug"],
  "required_tools": ["read_file", "list_dir"],
  "preferred_profile": "safe|balanced|power",
  "risk_level": "low|medium|high"
}
```

Access profiles grant least-privilege tool sets:
- **safe** — read-only (`read_file`, `list_dir`, `grep_search`, `semantic_search`)
- **balanced** — adds write ops (`apply_patch`, `create_file`, `run_in_terminal`)
- **power** — adds network + subagent spawning (`github_repo`, `runSubagent`)

## AgentX CLI Reference

```bash
agentx list                          # list all registered agents
agentx find <tag-or-keyword>         # search by tag or keyword
agentx check <agent-id> --profile <profile>   # validate tool access
agentx recommend <agent-id>          # suggest least-privilege profile
agentx import-agency <path> --merge  # import from external agency repo
```

## Rollout Script

`tools/rollout_all_repos.py` detects stack tags from each repo, generates custom agents, and writes:
- `.agentx/agents.json` + `access_profiles.json` + `agency_import.json` + `README.md`
- `.github/copilot-instructions.md` — per-repo Copilot custom instructions
- `.vscode/mcp.json` — MCP server configuration
- `.vscode/settings.json` — VS Code Copilot settings

## MCP Servers Available

The `.vscode/mcp.json` wires up the **GitHub MCP server**, giving Copilot tools to:
- Search and read issues, pull requests, and code across the org
- Create and update issues and PRs
- Fetch file contents and commit history

Use `#github` tool references in Copilot Chat to activate these capabilities.

## Preferred Patterns

- When modifying `rollout_all_repos.py`, keep `write_agentx_pack` and `write_copilot_files` as separate functions
- When adding new stack detection, extend `detect_stack_tags` and update `build_custom_agents` conditionals
- Keep per-repo generated content deterministic (no timestamps, no random IDs)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Unwrenchable)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Unwrenchable)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
