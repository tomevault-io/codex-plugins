# AGENTS.md

This file provides guidance to Agent when working with code in this repository.

## Project Overview

**toyoura-nagisa** is an AI agent platform for professional scientific computing, focusing on ITASCA PFC (Particle Flow Code) discrete element simulations with real-time WebSocket communication.

### Technology Stack

- **Backend**: Python 3.10+, FastAPI, uvicorn, ChromaDB
- **Frontend**: React 19, TypeScript, Material-UI, Vite
- **AI Infrastructure**: Multi-provider LLM support, real-time streaming, in-process tool orchestration (optional MCP gateway)
- **Scientific Computing**: WebSocket integration with ITASCA PFC Python SDK
- **Communication**: WebSocket for real-time updates, RESTful API for state management

## Architecture & Core Features

### Clean Architecture Pattern

```
Presentation Layer (API, WebSocket, Request Handlers)
    ↓ orchestrates through
Application Layer (Use Cases, Tooling, Agent Runtime)
    ↓ uses
Domain Layer (Models, Business Rules, Contracts)
    ↑ implemented by adapters in
Infrastructure Layer (LLM, MCP Gateway, Memory, OAuth, PFC, Storage, WebSocket)
```

**Key Principles**:
- **Dependency Inversion**: Infrastructure depends on domain abstractions
- **Single Responsibility**: Each layer has clear boundaries
- **Testability**: Domain logic isolated from external systems

**Layer Responsibilities**:
- **Presentation**: API routes, WebSocket handlers, request/response formatting
- **Application**: Business logic orchestration (ChatOrchestrator, content processing, tool execution)
- **Domain**: Core models and business rules (StreamingChunk, BaseMessage)
- **Infrastructure**: External integrations (LLM providers, optional MCP gateway, storage)

**Example**: Swapping LLM providers requires zero application/domain layer changes (`packages/backend/infrastructure/llm/base/client.py`)

### LLM Providers

Located in `packages/backend/infrastructure/llm/providers/`: google (primary), anthropic, moonshot, openai, openai_codex, openrouter, zhipu, google_antigravity, google_gemini_cli, web_search.

**Configuration**: `packages/backend/infrastructure/storage/llm_config_manager.py` (runtime provider/model selection)

### SubAgent System

MainAgent can delegate specialized tasks to lightweight SubAgents via the `invoke_agent` tool.

**Available SubAgents**:

| SubAgent | Tools | Max Iterations | Purpose |
|----------|-------|----------------|---------|
| **PFC Explorer** | 14 read-only | 64 | Documentation search, codebase exploration |
| **PFC Diagnostic** | 9 read-only | 64 | Multimodal visual analysis, task status inspection |

**PFC Explorer Tools**: `read`, `glob`, `grep`, `bash`, `bash_output`, `pfc_browse_commands`, `pfc_browse_python_api`, `pfc_query_python_api`, `pfc_query_command`, `pfc_browse_reference`, `pfc_list_tasks`, `pfc_check_task_status`, `web_search`, `todo_write`

**PFC Diagnostic Tools**: `pfc_capture_plot`, `read`, `pfc_check_task_status`, `pfc_list_tasks`, `glob`, `grep`, `bash`, `bash_output`, `todo_write`

**Architecture**:
- **Non-Streaming**: SubAgents use synchronous API calls for efficiency
- **Memory Isolation**: SubAgent todos stored in memory (`_memory_todos`), not persisted to disk
- **Context Separation**: SubAgent exploration doesn't consume MainAgent's context window
- **Read-Only**: SubAgents cannot execute simulations or modify files (no `pfc_execute_task`, `write`, `edit`)

**Storage Mode** (determined by agent type):
- MainAgent (`pfc_expert`): `persistent=True` → local file (`workspace/todos.json`)
- SubAgent (`pfc_explorer`, `pfc_diagnostic`): `persistent=False` → in-memory storage

**Implementation**:
- Tool: `packages/backend/application/tools/agent/invoke_agent.py`
- Config loader: `packages/backend/domain/models/agent_profiles.py` (source data in `config/agents.yaml`)
- Prompts: `packages/backend/config/prompts/pfc_explorer.md`, `packages/backend/config/prompts/pfc_diagnostic.md`

### Tool System

**In-process Tool Categories** (`packages/backend/application/tools/`):
- `coding/`: write, read, edit, bash, bash_output, kill_shell, glob, grep
- `pfc/`: pfc_execute_task, pfc_interrupt_task, pfc_check_task_status, pfc_list_tasks, pfc_capture_plot, pfc_query_*, pfc_browse_*
- `planning/`: todo_write
- `agent/`: invoke_agent, trigger_skill
- `builtin/`: web_search, web_fetch

**Optional MCP Gateway**:
- `packages/backend/infrastructure/mcp/` can expose internal tools to external MCP clients
- Internal tool execution does not require an MCP server

### PFC Integration Overview

**Problem**: AI-assisted control of professional scientific software (ITASCA PFC) requires solving:
1. Thread-safety (PFC SDK requires main thread execution)
2. Long-running tasks (simulations can run for hours)
3. Progress visibility (commands are non-interruptible)
4. Documentation-driven workflow (LLM needs command syntax guidance)

**Solution Architecture**:
```
Documentation Query → Test Script → Production Script → WebSocket → PFC SDK
     ↓                    ↓              ↓
Query Tools      Small-Scale Test   Full Simulation
(syntax ref)     (quick validate)   (task tracking)
```

**Key Components**:
- **Main Thread Executor**: Queue-based execution ensuring thread safety (`pfc-mcp` repo: `pfc-bridge/server/execution/main_thread.py`)
- **Task Manager**: Non-blocking lifecycle tracking (`pfc-mcp` repo: `pfc-bridge/server/tasks/manager.py`)
- **Script Executor**: Real-time output capture for progress monitoring (`pfc-mcp` repo: `pfc-bridge/server/execution/script.py`)
- **Interrupt/Diagnostic Signals**: Callback-based execution for non-blocking diagnostics during simulation cycles (`pfc-mcp` repo: `pfc-bridge/server/signals/`)
- **Documentation System**: Command syntax + Python usage examples (`pfc-mcp` repo: `src/pfc_mcp/docs/resources/`)

**PFC Tools Workflow (Script-Only)**:
1. **Query**: `pfc_query_command` / `pfc_query_python_api` - Get command syntax and Python examples
2. **Test**: `pfc_execute_task` (small scale, `run_in_background=False`) - Quick validation
3. **Production**: `pfc_execute_task` (full scale, `run_in_background=True`) - Long simulations with git snapshot
4. **Monitor**: `pfc_check_task_status` - Query progress with real-time output
5. **List**: `pfc_list_tasks` - Overview of all tracked tasks with version info

**Core Pattern**: All PFC commands executed via Python scripts using `itasca.command("...")` pattern.

**Git Version Tracking**:
- Each `pfc_execute_task` creates a git snapshot on the `pfc-executions` branch
- **IMPORTANT**: NEVER checkout or work on the `pfc-executions` branch in PFC projects
- This branch is automatically managed; working on it will break git snapshot creation
- If accidentally on this branch, switch back: `git checkout master`

**Detailed Documentation**: See `https://github.com/yusong652/pfc-mcp/tree/main/pfc-bridge` for implementation details, thread-safety architecture, and usage examples.

**Backend Integration**: `config/mcp.json` + `packages/backend/infrastructure/mcp/client.py`

## Development Commands

### Prerequisites
- **GitHub CLI**: Required for issue management and PR workflows
  ```bash
  # macOS
  brew install gh

  # Linux (Debian/Ubuntu)
  curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg \
  && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
  && sudo apt update \
  && sudo apt install gh

  # After installation, authenticate with:
  gh auth login
  ```

### Backend Development
```bash
# Start the backend server
npm run dev:backend
# OR manually:
cd packages/backend && uv run python run.py

# Internal MCP server module has been removed.
# Use external MCP servers via config/mcp.json (for example: uvx pfc-mcp).

# Run tests
uv run pytest

# Install dependencies
uv sync

# Install development dependencies (including GitHub CLI)
uv sync --extra dev
```

### Frontend Development
```bash
# Start Web frontend
npm run dev:web
# OR manually:
cd packages/web && npm run dev

# Build for production
npm run build

# Run linting
npm run lint

# Preview production build
npm run preview
```

### PFC Integration Development

**Important**: `pfc-mcp` / `pfc-bridge` now live in a standalone repository: `https://github.com/yusong652/pfc-mcp`.

```bash
# 1. Clone standalone pfc-mcp repository
git clone git@github.com:yusong652/pfc-mcp.git

# 2. Install pfc-bridge dependencies in PFC's Python environment
#    (Run in PFC GUI IPython console)
pip install websockets==9.1

# 3. Start PFC WebSocket server (in PFC GUI Python console)
exec(open(r'C:\Dev\Han\pfc-mcp\pfc-bridge\start_bridge.py', encoding='utf-8').read())

# 4. Verify integration from toyoura-nagisa (with PFC server running)
#    Use PFC tools in app/backend (e.g., pfc_list_tasks, then a small pfc_execute_task)
```

**Why separate?**
- pfc-bridge requires `websockets==9.1` (PFC Python environment constraint)
- toyoura-nagisa requires `websockets>=15.0.1` (modern features)
- Different runtime environments → separate dependency management

## File Structure

```
toyoura-nagisa/
├── config/
│   └── agents.yaml                  # Main + SubAgent definitions
├── packages/
│   ├── backend/
│   │   ├── app.py                   # Main FastAPI application
│   │   ├── presentation/            # API + WebSocket entry layer
│   │   │   ├── api/
│   │   │   ├── websocket/
│   │   │   ├── handlers/
│   │   │   ├── models/
│   │   │   └── exceptions/
│   │   ├── application/             # Use cases and orchestration
│   │   │   ├── agent/
│   │   │   ├── chat/
│   │   │   ├── contents/
│   │   │   ├── notifications/
│   │   │   ├── oauth/
│   │   │   ├── pfc/
│   │   │   ├── reminder/
│   │   │   ├── session/
│   │   │   ├── shell/
│   │   │   ├── skills/
│   │   │   ├── todo/
│   │   │   └── tools/               # coding/pfc/planning/agent/builtin/runtime
│   │   ├── domain/
│   │   │   ├── models/
│   │   │   └── utils/
│   │   ├── infrastructure/          # External integrations
│   │   │   ├── llm/
│   │   │   │   ├── base/
│   │   │   │   ├── providers/       # google, anthropic, openai, openai_codex, moonshot, openrouter, zhipu, ...
│   │   │   │   └── shared/
│   │   │   ├── file_mention/
│   │   │   ├── mcp/
│   │   │   ├── messaging/
│   │   │   ├── monitoring/
│   │   │   ├── oauth/
│   │   │   ├── pfc/
│   │   │   ├── shell/
│   │   │   ├── skills/
│   │   │   ├── storage/
│   │   │   ├── web_fetch/
│   │   │   └── websocket/
│   │   ├── config/
│   │   │   └── prompts/
│   │   ├── shared/                  # constants/exceptions/utils
│   │   └── workspace/
│   ├── web/                         # React web frontend
│   ├── cli/                         # React/Ink terminal frontend
│   ├── core/                        # Shared TypeScript core
│   ├── credentials/
│   ├── pfc_workspace/
│   └── workspace/
├── data/                            # Session data + oauth tokens
├── tests/                           # Root test suite
├── workspace/                       # Runtime workspace
├── package.json                     # Root package.json (npm workspaces)
└── pyproject.toml                   # Root Python configuration (uv workspace)
```

Standalone dependency:
- `https://github.com/yusong652/pfc-mcp` (contains `pfc-mcp` MCP server + `pfc-bridge` runtime)

## Configuration

### Environment Setup
- Backend configuration lives in `packages/backend/config/` (version-controlled)
- Main config files: `cors.py`, `dev.py`, `pfc.py`
- Agent definitions: `config/agents.yaml`
- Database locations:
  - Session data: `data/` (root level)

### Google Services Integration
Many tools integrate with Google services via OAuth:
- Authentication tokens stored in `data/oauth_tokens/google/`
- Supports Gmail, Google Calendar, and Google Contacts
- OAuth flow is managed through backend API endpoints in `packages/backend/presentation/api/oauth.py`

## CLI Commands

- **Shell** (`!` prefix): `! git status` - Execute bash, results injected to LLM context
- **PFC Console** (`>` prefix): `> ball.count()` - Execute in PFC Python environment

## Testing

### Backend Testing
```bash
# Run all tests
uv run pytest
```

### Frontend Testing
The frontend uses standard React testing practices with Vite.

### PFC Integration Testing
```bash
# Start PFC server first, then verify with tool calls:
# 1) pfc_list_tasks        (connectivity and task store)
# 2) pfc_execute_task      (small foreground/background script)
# 3) pfc_check_task_status (progress and completion states)
```

## Common Issues & Quick Fixes

### LLM Provider Selection
- Configure runtime provider/model via `packages/backend/infrastructure/storage/llm_config_manager.py`
- Provider config API: `packages/backend/presentation/api/llm_config.py`
- Frontend session-level updates are persisted to `chat/data/<session_id>/metadata.json` under `llm_config`; new session defaults come from `config/models.yaml`
- All providers support tool calling and streaming

### Tool Loading
- Tools are loaded dynamically based on agent name (main agent or SubAgent)
- Main agent uses a single configuration; SubAgents are invoked explicitly
- Internal tools run in-process; MCP gateway is optional for external MCP clients (port 9000 if enabled)

### PFC Integration
- **Not a workspace member**: pfc-bridge runs in PFC's Python environment
- **Dependency installation**: Run `pip install websockets==9.1` in PFC GUI
- **Server port**: Runs on port 9001 (WebSocket)
- **Startup**: Must be started in PFC GUI before using PFC tools
- **Long tasks**: Return task_id immediately for non-blocking operation
- **Troubleshooting**: Check `https://github.com/yusong652/pfc-mcp/tree/main/pfc-bridge`

## Code Modification Guidelines

When using batch commands (sed, find, etc.):
- Consider the context: Will regex affect type annotations, decorators, or critical syntax?
- Verify results: Check `git diff` samples and run imports/type checks on modified files
- For Pydantic models and base classes: prefer manual Edit to preserve structure

## Related Guides

- `.claude/guides/typescript-guide.md` - TypeScript/React patterns
- `.claude/guides/code-standards.md` - Code quality standards
- `https://github.com/yusong652/pfc-mcp/tree/main/pfc-bridge` - PFC integration details

## Git Commit Format

```
feat: brief description

Co-authored-with: Nagisa Toyoura <nagisa.toyoura@gmail.com>
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yusong652) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
