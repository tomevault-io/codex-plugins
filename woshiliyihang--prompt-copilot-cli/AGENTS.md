# Copilot instructions for prompt-copilot-cli

This file gives Copilot sessions repository-specific guidance to speed accurate, low-risk edits and tasks.

1) Build, test, and release
- Build (local artifact): python -m build
- Check built artifacts: python -m twine check dist/*
- Publish to PyPI (used by repository script): python scripts\release.py --skip-upload  # validates build
  - Full release (interactive / token): python scripts\release.py
- Run tests (full suite): python -m pytest -q
- Run a single test file: python -m pytest tests/test_cli_args.py -q
- Run a single test case by name: python -m pytest tests/<file>::<test_name> -q
- Alternative single-test selection: python -m pytest -k "substring_of_test_name" -q

(There is no centralized lint config in the repo; do not invent or add linter tooling without explicit PR.)

2) High-level architecture (big picture)
- main.py: the CLI entry point and console script (console script: prompt-copilot). It parses args and drives the interactive agent.
- Core capabilities are implemented as tools exposed to the interactive CLI: file operations, command execution, Python script execution, and an image->base64 helper for multimodal models.
- MCP integration: the agent discovers and calls external MCP servers configured in the user config (see config locations below). MCP servers supply additional tools via the MCP protocol.
- Packaging & release: scripts/release.py reads APPLICATION_VERSION from main.py, updates pyproject.toml, builds artifacts with PEP517 tooling, checks with twine, and uploads using a PyPI token.
- Tests: located under tests/ and run with pytest.

3) Key repo-specific conventions
- Version authoritative source: APPLICATION_VERSION in main.py. scripts/release.py syncs pyproject.toml to that value before building.
- Console script: project defines prompt-copilot in pyproject (entry point main:main). Use that name for manual testing of installed package.
- Config file (user-level):
  - Windows: %USERPROFILE%\.prompt-copilot\config.json
  - Unix: ~/.prompt-copilot/config.json
  - Relevant keys: "model", "base_url", "api_key", "mcp" (enabled + servers array)
- MCP discovery: mcp.servers in the config is the mechanism to provide external tools. Provide server objects matching MCP conventions.
- Packaging: repo requires Python 3.10–3.14 (pyproject: requires-python >=3.10,<3.15).

4) Useful operational hints for Copilot sessions
- Do not change version in pyproject.toml directly; update APPLICATION_VERSION in main.py and use scripts/release.py to propagate.
- For quick local checks, run the console script against a temporary directory: prompt-copilot -d <path> -t "<task>".
- When editing code that affects packaging or release flow, run python -m build and python -m twine check locally to validate artifacts before pushing.

5) AI/assistant integration files
- No repository-level Copilot-specific file was present prior to this commit; include config here so future sessions have a consistent baseline.

Summary: created .github/copilot-instructions.md capturing build/test commands, architecture overview, and repo conventions. Want any additions (e.g., explicit test names to run, or an example MCP server config to add to README)?

---
> Source: [woshiliyihang/prompt-copilot-cli](https://github.com/woshiliyihang/prompt-copilot-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-07 -->
