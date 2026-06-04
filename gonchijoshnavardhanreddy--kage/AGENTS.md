# Kage - Copilot Instructions

AI-powered penetration testing CLI assistant for bug bounty, red team, and CTF work.

## Build & Test Commands

```bash
# Install with dev dependencies
pip install -e ".[dev]"

# Run all tests
pytest tests/ -v

# Run a single test file
pytest tests/unit/test_models.py -v

# Run a specific test
pytest tests/unit/test_models.py::test_name -v

# Run with coverage
pytest tests/ -v --cov=src/kage --cov-report=html

# Linting
ruff check src/ tests/
mypy src/kage/

# Format code
ruff format src/ tests/
ruff check --fix src/ tests/
```

## Architecture

Kage uses a layered architecture with clear separation of concerns:

```
CLI Layer (cli/)           → Typer commands, Rich UI components, setup wizard
    ↓
Core Engine (core/)        → Conversation manager, command orchestrator, session state
    ↓
AI Provider (ai/)          → LLM abstraction with providers: Ollama, OpenAI, LM Studio
    ↓
Execution Engine (executor/) → Local shell, SSH, Docker, WSL executors
    ↓
Security Layer (security/)  → Scope validation, safe mode, user approval, audit logging
    ↓
Plugin System (plugins/)    → Plugin discovery, loading, sandboxed execution
    ↓
Reporting (reporting/)      → Findings management, template engine (Jinja2), export
    ↓
Persistence (persistence/)  → Session store (JSON), config (YAML), audit logs (JSONL)
```

### Key Data Flow

All commands flow through a security pipeline before execution:
```
AI Output → Scope Validator → Safe Mode Filter → User Approval → Execution → Audit Log
```

### Core Models (`core/models.py`)

- `Session` - Root aggregate containing scope, messages, commands, findings
- `Command` - Represents a command with status workflow (pending → approved → running → completed)
- `Finding` - Security finding with CVSS severity, CWE, evidence
- `AuditEntry` - Tamper-evident log entry with hash chain

## Key Conventions

### Async-First Design
- Command execution and LLM calls are async (`async/await`)
- Use `pytest-asyncio` with `asyncio_mode = "auto"` for async tests

### Pydantic Models
- All data structures use Pydantic v2 with strict typing
- Models have `Field(default_factory=...)` for mutable defaults
- Config uses `pydantic-settings` for environment variable support

### Plugin System
- Plugins live in `plugins/<name>/` with `plugin.yaml` metadata
- Inherit from `KagePlugin` base class in `kage.plugins.base`
- Use `@capability` decorator to expose functions to the AI
- Plugins run in a restricted sandbox (no subprocess, socket, eval)

### Security Enforcement
- `dangerous=True` on capabilities/commands requires user approval
- Scope validation checks all IPs, CIDRs, domains in commands
- Safe mode blocks destructive patterns (`rm -rf /`, fork bombs, etc.)
- Audit log uses hash chain for tamper detection

### CLI Structure
- Main entry: `kage.cli.app:main`
- Commands in `cli/commands/` subdirectory
- Rich console for formatted output

### Type Hints
- Strict mypy enforcement (`strict = true`)
- Use `str | None` syntax (Python 3.10+)
- All public functions must have type annotations

---
> Source: [GonchiJoshnaVardhanReddy/Kage](https://github.com/GonchiJoshnaVardhanReddy/Kage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-04 -->
