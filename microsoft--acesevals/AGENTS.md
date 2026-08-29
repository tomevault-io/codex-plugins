```instructions
# ACES / SABER Repository – Copilot Instructions

> **Naming:** The external name for this project is **ACES** (Agent Capability Evaluation Suite). **SABER** (Security Agent Benchmarking and Evaluation Research) is the internal Microsoft codename. The Python package, CLI commands, and code all use the name `saber`. Both names refer to the same system.
>
> **Repositories (GitHub is the permanent home):**
> - **GitHub (permanent home):** [ACESEvals](https://github.com/microsoft/ACESEvals) (benchmarks) + [ACES](https://github.com/microsoft/ACES) (library)
> - **Azure DevOps (deprecated — read-only after 2026-09-30):** [oss_saber](https://dev.azure.com/MSECAIModels/Benchmarking/_git/oss_saber) + [SABER](https://dev.azure.com/MSECAIModels/Benchmarking/_git/SABER). Being retired; do new work on GitHub.

**SABER** is a distributed system for benchmarking agentic workflows in cybersecurity domains using **inspect_ai** integration with **Model Context Protocol (MCP)**.

## Repository Structure

| Path | Purpose |
|------|---------|
| `src/saber/server/` | FastAPI + FastMCP server – REST API, session management, Docker sandbox execution |
| `src/saber/client/` | Client-side – session management, REST client, MCP tool access |
| `src/saber/inspect_ai/` | Inspect AI integration – SABERSandboxEnvironment, agent registry, solvers |
| `src/saber/domain/` | Domain orchestration – manifest loading, Docker compose management |
| `src/saber/models/` | Shared Pydantic models – REST, MCP, evaluation, benchmark task models |
| `domains/` | Benchmark domains – task configurations, prompts, Docker environments |
| `notebooks/` | Analysis notebooks – model comparison, agent architecture analysis, cross-domain aggregation |
| `docs/` | Architecture documentation and diagrams |
| `tests/` | Test suite organized by component |

## Architecture Overview

```
Inspect AI → SABERSandboxEnvironment → ClientSessionManager → REST/MCP Server
                      ↓                        ↓                    ↓
               Agent Solver              MCP Client           SessionManager
                      ↓                        ↓                    ↓
               Tool Execution          Tool Discovery      Docker Sandbox Execution
```

**Key Components**:
- `SABERSandboxEnvironment`: Inspect AI sandbox lifecycle management (task_init, sample_init, sample_cleanup)
- `SessionManager`: Central server orchestrator – sessions, episodes, execution, evaluation
- `ClientSessionManager`: Client-side session and episode management via REST
- `ExecutionManager`: Docker sandbox command execution with security validation
- `EvaluationManager`: Client-side evaluation result storage and validation
- `BenchmarkManager`: YAML-based task and benchmark configuration loading

**Key APIs**:
- REST: `/sessions`, `/episodes`, `/benchmarks`, `/policy`, `/evaluation`
- MCP: Tool discovery and execution via FastMCP server

## Python Environment

- **Use `uv`** – the package manager for this project
- Run commands: `uv run pytest`, `uv run pre-commit`
- Install deps: `uv sync --all-extras` (workspace root)
- **Do NOT** use system Python or create new virtual environments
- Python 3.11-3.12 required (managed by `.python-version`)

## Dependency Management — saber Library

The `saber` library (from the ACES/SABER repo) is installed via git in `pyproject.toml`
using `branch = "main"`, but `uv.lock` pins the **exact commit hash**. This means:

- `uv sync` installs whatever commit is locked, **not** the latest on `main`.
- After changes are pushed to the ACES/SABER library repo, the lock here must be
  explicitly updated:
  ```bash
  uv lock --upgrade-package saber   # resolves latest commit on main
  uv sync --all-extras               # installs it
  ```
  Then commit the updated `uv.lock`.
- **If evals produce unexpected errors** (missing features, broken scoring), the first
  thing to check is whether `uv.lock` is pointing at a stale saber commit.
- Use `uv pip show saber` to see the currently installed commit hash.

> **`inspect-ai` is a REQUIRED fork — do not replace with upstream.** `pyproject.toml`
> pins `inspect-ai` to the GitHub `ACESEvals` branch `inspect-ai/dev/aces_integration`.
> This fork powers the agent harnesses (`-T agent=copilot`, `-T agent=claude_code`) and
> tool_call limits, which are **not** in upstream inspect_ai. Switching to upstream breaks
> the harness / agent-architecture evals (see `notebooks/*agent_architecture_analysis.ipynb`).
> This branch must keep being maintained on GitHub; it is not retired by the ADO deprecation.

> **Agent harnesses run their CLIs inside the sandbox, not on the host.** `-T agent=copilot`
> and `-T agent=claude_code` use inspect_ai's `sandbox_agent_bridge()` to run the GitHub
> Copilot / Claude Code CLIs **inside the Docker sandbox** — the base `saber/sandbox` image
> ships Node 22 + the `claude` CLI + the Copilot SDK — with model calls proxied back to
> inspect_ai's `--model` (BYOK). The **host** only needs the Python SDKs (`uv sync
> --all-extras` installs `github-copilot-sdk` + `claude-code-sdk`); host-side Node is **not**
> required. If a harness fails with a missing `claude`/`copilot` runtime, rebuild the sandbox
> image rather than installing anything on the host.

## Code Quality

- **Pre-commit**: Ruff linting/formatting (see `.pre-commit-config.yaml`)
- **Type hints**: Required on all public functions
- **Docstrings**: Args/Returns/Raises sections for public APIs
- **Clean code**: Small focused functions, DRY, meaningful names, proper error handling
- **Pydantic v2**: Use `ConfigDict(frozen=True)` for immutable data models

## Testing

- **Framework**: pytest with markers
- **Unit tests**: `uv run pytest` (default, no external deps)
- **Integration**: `uv run pytest -m integration` (requires Docker/services)
- **E2E**: `uv run pytest -m e2e` (full stack)
- Use `tmp_path` fixture, mock Docker/network calls
- Coverage: `uv run coverage run -m pytest && uv run coverage report`

## Key Domain Concepts

- **Session**: Client connection with lifecycle management and multiple episodes
- **Episode**: Single evaluation instance in an isolated Docker sandbox
- **Task**: YAML-defined benchmark scenario with prompts, subtasks, and evaluation config
- **Benchmark**: Collection of related tasks in a domain (excytin_demo, airt_demo)
- **Sandbox**: Docker Compose environment for secure command execution
- **MCP Tools**: Model Context Protocol tools exposed to agents for task execution
- **Evaluation**: Client-side scoring (static, LLM-judge, tool-call strategies)

## Logging Categories

SABER uses structured logging with categories:
- `LogCategory.SESSION_MANAGER`: Session and episode lifecycle
- `LogCategory.EXECUTION`: Command execution in sandboxes
- `LogCategory.EVALUATION`: Evaluation and scoring
- `LogCategory.AGENT`: Agent solver and tool execution
- `LogCategory.TASK_MANAGER`: Task and benchmark loading

## Local Development

```bash
# Install dependencies
uv sync --all-extras

# Run unit tests
uv run pytest tests/

# Run specific test file
uv run pytest tests/server/test_session_manager.py -v

# Run quality checks
uv run pre-commit run --all-files

# Enable detailed logging for debugging
INSPECT_LOG_LEVEL=info uv run inspect eval domains/excytin_demo --model openai/gpt-4
```

## Running Evaluations

```bash
# Basic evaluation
uv run inspect eval domains/excytin_demo --model openai/gpt-4

# With task filtering
uv run inspect eval domains/excytin_demo --model openai/gpt-4 -T task_filter="incident_*"

# Build/rebuild Docker images
uv run inspect eval domains/excytin_demo --model openai/gpt-4 -T build=true
uv run inspect eval domains/excytin_demo --model openai/gpt-4 -T rebuild_all=true

# Concurrency control
uv run inspect eval domains/excytin_demo --model openai/gpt-4 --max-samples 4 --max-connections 20
```

> **Reasoning models — always set `--reasoning-effort`.** gpt-5.x / o1 / o3 / Claude Opus
> do **no reasoning by default** here (`reasoning_effort` unset ⇒ 0 reasoning tokens ⇒
> lower scores). Pass it explicitly, e.g. `--reasoning-effort high`. Verified: excytin
> gpt-5.4 `latest_test_set` scored 0.813 (unset) vs 0.886 (high) — a +0.073 swing driven
> entirely by reasoning. Always fix reasoning effort before comparing runs, and suspect it
> first if scores drift.

## Key Task Parameters (-T flags)

| Parameter | Purpose |
|-----------|---------|
| `task_filter` | Filter tasks by name/pattern |
| `build=true` | Build missing Docker images |
| `rebuild_all=true` | Rebuild all Docker images |
| `rebuild=<prefix>` | Rebuild specific images by prefix |
| `stop_saber_after=true` | Stop server after evaluation |
| `run_preflight=true` | Run health checks before evaluation |
| `rest_port` / `mcp_port` | Custom server ports |
| `agent=<name>` | Specify agent implementation |

## Evaluation Analysis

The `notebooks/` directory contains a comprehensive data science analysis framework for comparing models and agent architectures across all SABER domains.

### Analysis Notebooks

| Notebook | Purpose |
|----------|---------|
| `notebooks/eval_analysis.ipynb` | Domain-agnostic analysis template (14 experiments) |
| `notebooks/{domain}_analysis.ipynb` | Self-contained per-domain analysis (excytin, cybench, cti_realm) |
| `notebooks/{domain}_agent_architecture_analysis.ipynb` | Agent architecture comparison per domain |
| `notebooks/aggregate_model_analysis.ipynb` | Cross-domain model comparison (domain-normalized) |
| `notebooks/aggregate_agent_architecture_analysis.ipynb` | Cross-domain agent architecture comparison |

### Shared Analysis Library

`notebooks/saber_analysis/` provides reusable utilities: `load_eval_logs()`, `load_trajectory_data()`, `extract_cost_rows()`, `setup_plotting()`. Use these for programmatic analysis outside notebooks.

### Analysis Artifacts

Generated visualizations are saved to `notebooks/artifacts/{domain}/` — overall reward, cost analysis, token usage, sub-task breakdowns, effort distributions, tool usage, and more.

### Key Analysis Concepts

- **14 standard experiments**: Overall reward, per-group breakdown, cost, tokens, cost efficiency, sub-task decomposition, checkpoint distributions, gap heatmaps, trajectory analysis, effort budget, tool usage, time efficiency, effort segmentation, difficulty agreement
- **Domain-normalization**: Cross-domain aggregation uses equal weight per domain (prevents sample-rich domains from dominating)
- **Agent architectures**: React, GH Copilot, Claude Code compared for fixed model (Sonnet 4.6)

For full analysis methodology, see the `eval-analysis` skill (`.github/skills/eval-analysis/SKILL.md`).

## Common Patterns

### Creating a New Task
1. Add task YAML in `domains/<domain>/server/config/tasks/`
2. Include required prompts: `instruction`, `assistant`, `submit`, `continue`
3. Define `environment` (Docker Compose reference or inline)
4. Configure `submission_evaluation_config` and optionally `step_evaluation_config`

### Adding a New Agent
1. Create agent in `src/saber/inspect_ai/agents/registry/` or domain-local `client/`
2. Register with `@SABERAgentRegistry.register("agent_name")`
3. Implement `create_agent(domain_slug, ...)` function returning `Solver`

### Server-Side Component Changes
1. Update models in `src/saber/models/`
2. Implement logic in appropriate manager (`SessionManager`, `ExecutionManager`, etc.)
3. Add REST endpoint in `src/saber/server/api/`
4. Add tests in `tests/server/`

**Documentation**: [docs/README.md](docs/README.md) | **Domains**: [domains/](domains/) | **Architecture**: [docs/assets/](docs/assets/)

```

---
> Source: [microsoft/ACESEvals](https://github.com/microsoft/ACESEvals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-29 -->
