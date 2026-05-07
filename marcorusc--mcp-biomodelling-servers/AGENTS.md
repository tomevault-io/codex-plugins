# AI Coding Agent Instructions for MCP Bio-Modelling Servers

## Project Overview
This repository provides **Model Context Protocol (MCP) servers** for systems biology modeling tools: **NeKo** (network construction), **MaBoSS** (Boolean dynamics), and **PhysiCell** (multiscale tissue simulation). Each server exposes Python modeling libraries as MCP tools for LLM-driven workflows.

**Key Architecture**: Three independent MCP servers (`MaBoSS/`, `NeKo/`, `PhysiCell/`) that:
- Share a single Conda environment but run as separate stdio processes
- Communicate via cross-server workflows (e.g., NeKo exports BNET → MaBoSS simulates → PhysiCell integrates)
- Use session-based state management (not stateless tools)
- Follow a "guidance over errors" pattern for enhanced LLM interaction

## Critical Workflow: Cross-Server Pipeline
The **recommended end-to-end workflow** is documented in server docstrings and READMEs:
1. **NeKo**: Build gene regulatory network from gene lists → export as BNET
2. **MaBoSS**: Convert BNET → BND/CFG → run Boolean simulation → analyze attractors
3. **PhysiCell**: Load MaBoSS results → link Boolean nodes to cell behaviors → run multiscale simulation

**Why this matters**: Tools are designed for **chaining**, not isolation. Always check server docstrings for cross-references (e.g., `NeKo/server.py` line 1-24, `PhysiCell/README.md` lines 16-28).

## Session Management Pattern (NeKo & PhysiCell)
**Critical convention**: Both NeKo and PhysiCell use **session-based state** to prevent accidental reuse of old networks/configs:

```python
# ALWAYS create session first (prevents stale state bugs)
create_session()  # Returns session_id
set_default_params(max_len=2, only_signed=True)  # Optional tuning
create_network(['TP53', 'MYC'], session_id=session_id)  # Use explicit session_id
```

**Implementation**: See `NeKo/session_manager.py` and `PhysiCell/session_manager.py`:
- `NeKoSession` dataclass: stores `network` object + `_edges_cache` (invalidated on mutations)
- `SessionState` dataclass: stores `PhysiCellConfig` + `completed_steps` (workflow tracking)
- Thread-safe: uses `threading.Lock` for concurrent access
- `ensure_session(session_id)`: returns default session if `session_id=None`

**Why sessions?**: Multi-hypothesis testing (LLMs often explore alternatives in parallel).

## Code Conventions

### 1. Error Handling: Guidance Over Exceptions
**Pattern**: Return Markdown-formatted guidance strings instead of raising exceptions. See `NeKo/utils.py`:

```python
def format_connectivity_guidance() -> str:
    """Provide guidance when network isn't connected for BNET export."""
    return """## BNET Export Failed: Network Not Connected
    
**Solutions to improve connectivity:**
1. Expand pathway length: `create_network(genes, max_len=3)`
2. Include unsigned interactions: `only_signed=False`
3. Add hub genes: `add_gene('TP53')`"""
```

**Used in**: All `format_*_guidance()` functions (8 variants in `NeKo/utils.py`). This helps LLMs self-correct without breaking tool chains.

### 2. Verbosity Levels (NeKo-specific)
Control output length to manage token costs in LLM loops:
- `summary`: Minimal status (e.g., "Network created: nodes=50 edges=120")
- `preview`: Truncated tables (first 50 rows)
- `full`: Complete output (first 100 rows for large datasets)

**Implementation**: `session_manager.py` line 15-16, `normalize_verbosity()` function. Always default to `summary` in tool signatures.

### 3. Decorator Pattern for Network Requirements
```python
@requires_network  # Custom decorator, NeKo/server.py line 103
def add_gene(gene: str, session_id=None, sess=None, network=None):
    # Decorator injects sess, network; returns E_NO_NET if missing
```

**Why**: DRY principle for 30+ tools needing network validation. See `_session_network()` helper (line 94).

### 4. Markdown Table Formatting
Always use `clean_for_markdown()` (NeKo/utils.py line 198) before `.to_markdown()` to escape special characters:

```python
df['References'] = df['References'].apply(short_refs)  # Truncate first
md = clean_for_markdown(df).to_markdown(index=False, tablefmt="plain")
```

**Why**: Prevents Markdown rendering issues in LLM responses (especially pipe characters in gene names).

## MCP Tool Registration
All tools use `@mcp.tool()` decorator from `mcp.server.fastmcp.FastMCP`:

```python
from mcp.server.fastmcp import Context, FastMCP
mcp = FastMCP("NeKo")  # Server name

@mcp.tool()
def create_network(list_of_initial_genes: List[str], ctx: Context, ...) -> str:
    ctx.info(f"Creating network...")  # Logs to MCP client
    return "Markdown-formatted result"
```

**Context parameter**: Optional `ctx: Context` provides `.info()`, `.error()`, `.warning()` for structured logging.

## File Organization Patterns

### Export Directories
**Convention**: Each server writes to its own `exports/` subfolder (not workspace root):
```python
def _export_dir() -> Path:
    base = Path(__file__).parent  # Anchor to server.py location
    d = base / "exports"
    d.mkdir(exist_ok=True)
    return d
```

See `NeKo/server.py` line 87, `NeKo/exports/` folder (Network.bnet, Network.sif).

### File Cleanup Tools
All servers provide `clean_generated_files()` tools:
- **NeKo**: Removes `*.bnet` from `exports/`
- **MaBoSS**: Removes `output.bnd`, `output.cfg`
- **PhysiCell**: Removes XML exports (not yet implemented)

## Testing & Debugging

### How to Test Servers Locally
```bash
# 1. Activate Conda environment (all dependencies shared)
conda activate mcp_modelling

# 2. Run server directly (stdio mode)
cd NeKo
python server.py
# Paste JSON-RPC requests to stdin (or use MCP client like VS Code)

# 3. Check logs
cat ~/pypath_log/*.log  # NeKo dependency (pypath) logs here
```

### Common Integration Issues
1. **"No network in session"**: Forgot `create_session()` → `create_network()` order
2. **BNET export fails**: Network not connected → use `check_disconnected_nodes()`, `list_components()`, `apply_strategy('complete_connection')`
3. **Special characters in gene names**: BNET export auto-cleans with `re.sub(r"[^A-Za-z0-9_]", "_", name)` (NeKo/server.py line 468)
4. **Stale cache errors**: Sessions invalidate `_edges_cache` via `_invalidate(sess)` after mutations

## Key Files to Reference

### Architecture/Workflows
- `README.md`: MCP setup, Conda environment, VS Code configuration
- `NeKo/server.py` lines 1-24: Recommended NeKo→MaBoSS→PhysiCell workflow
- `PhysiCell/session_manager.py` lines 11-32: `WorkflowStep` enum (tracks simulation build progress)

### Session Management
- `NeKo/session_manager.py`: Lightweight session class with edge caching
- `PhysiCell/session_manager.py`: Advanced session with workflow tracking, MaBoSS context integration

### Error Handling
- `NeKo/utils.py`: 8 `format_*_guidance()` functions (guidance strings for common errors)

### Tool Examples
- `NeKo/server.py` line 159: `create_network()` — complex parameter handling, case-based logic
- `MaBoSS/server.py` line 98: `build_simulation()` — async tool with global state
- `PhysiCell/server.py` line 91: `create_session()` — session manager integration

## Dependencies
Install via Conda environment (see main README.md):
- **NeKo**: `nekomata` (network construction), `paramiko` (SSH support)
- **MaBoSS**: `maboss` (Boolean simulation), `conda-package-handling`
- **PhysiCell**: `physicell-settings>=0.3.0` (XML config builder)
- **Shared**: `fastmcp`, `pandas`, `matplotlib`

## What NOT to Do
1. **Don't create files in workspace root** — use `_export_dir()` pattern
2. **Don't raise exceptions in tool functions** — return guidance strings
3. **Don't assume global state** — always use sessions or check `if sim is None` (MaBoSS pattern)
4. **Don't use generic error messages** — provide actionable next steps (see utils.py)
5. **Don't ignore verbosity** — default to `summary` to reduce token costs in LLM loops

## Quick Reference: Common Tasks

### Add a new NeKo tool
1. Add `@mcp.tool()` decorated function to `server.py`
2. If needs network, use `@requires_network` decorator
3. Accept `session_id: Optional[str]` parameter
4. Return Markdown-formatted string
5. Update `get_help()` and `get_help_json()` tool lists

### Modify session state
1. Use `ensure_session(session_id)` to get/create session
2. Mutate session attributes (e.g., `sess.network = new_network`)
3. Call `_invalidate(sess)` if edges changed
4. Update `last_accessed` via `sess.touch()`

### Add cross-server integration
1. Document in server docstring (lines 1-30)
2. Add example to tool-specific README
3. Test file handoff (e.g., BNET export → MaBoSS import)
4. Update workflow guide tools (`workflow_guide()`)

---
> Source: [marcorusc/mcp-biomodelling-servers](https://github.com/marcorusc/mcp-biomodelling-servers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-07 -->
