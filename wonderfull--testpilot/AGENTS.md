# LangGraph Multi-Agent Requirement System - Copilot Instructions

This document provides context for GitHub Copilot and other AI assistants working on this codebase.

## Project Overview

- **Name:** LangGraph Multi-Agent Requirement System
- **Type:** Python/TypeScript multi-agent pipeline
- **Purpose:** Analyze software requirements and generate Gherkin BDD features + Playwright automation
- **Key Technologies:** LangGraph, OpenAI, Groq, SQLite (MCP-based), Playwright, Pydantic

## Architecture Summary

### Pipeline Flow

```
Input (Jira/File/Text)
  ↓ [Ingestion] → fetch via MCP tools
  ↓ [Change Detection] → semantic diff via Groq
  ↓ (skip if wording-only)
  ↓ [Classification] → area via OpenAI
  ↓ [Feature Gen] → Gherkin via OpenAI
  ↓ [Coverage Eval] → assessment via Groq
  ↓ [Router] → decide specialist path
  ├─ frontend? → [Playwright Agent]
  └─ other → [Post-Eval Agent]
  ↓ [Post-Eval] → QA via Groq
  ↓
Output (Features, Coverage, Automation)
```

### MCP Tool Usage Pattern

**All IO operations must use MCP tools, not direct calls:**

```python
# ✅ CORRECT: Use MCP tool abstraction
from services.db import get_db_service
db = get_db_service()
db.create_or_update_requirement(...)

# ❌ WRONG: Direct sqlite3 call
import sqlite3
conn = sqlite3.connect("db/context.db")
```

**MCP tools required:**
- `filesystem`: File read/write
- `sqlite`: Database operations
- `http` / `jira`: Jira REST API
- `shell`: npm, playwright commands
- `pdf_extract` / `docx_extract`: Document parsing

## Key Files & Responsibilities

| File | Responsibility |
|------|-----------------|
| `main.py` | CLI entrypoint using Typer |
| `graph.py` | LangGraph StateGraph orchestration |
| `state.py` | Pydantic state models & configs |
| `services/db.py` | MCP SQLite abstraction layer |
| `services/jira_client.py` | MCP HTTP/Jira tool wrapper |
| `services/nlp_utils.py` | Text processing (normalize, diff, extract) |
| `services/logging_utils.py` | Structured logging + LangSmith |
| `agents/*.py` | Individual agent nodes |

## Code Patterns & Conventions

### 1. Agent Structure

Every agent follows this pattern:

```python
class MyAgent(LoggingMixin):
    def __init__(self):
        super().__init__("my_agent")
        self.db_service = get_db_service()
    
    def process(self, state: RunState) -> RunState:
        self.log_info("Starting", run_id=state.run_id, requirement_id=state.requirement_id)
        # 1. Read from state
        # 2. Call MCP tools for IO
        # 3. Use LLM if needed (OpenAI or Groq)
        # 4. Update state
        # 5. Persist to SQLite if needed
        self.log_info("Complete", run_id=state.run_id)
        return state

def agent_node(state: RunState) -> RunState:
    if state.status != "expected_status":
        return state
    agent = MyAgent()
    return agent.process(state)
```

### 2. Logging Pattern

Always use LoggingMixin:

```python
self.log_info(
    "Message",
    run_id=state.run_id,
    requirement_id=state.requirement_id,
    metadata={"key": "value"}
)

self.log_error(
    "Error message",
    error=exception,
    run_id=state.run_id
)
```

### 3. MCP Tool Calls

All MCP tool calls go through service abstraction:

```python
# Database
db = get_db_service()
db.create_or_update_requirement(...)
db.create_artifact(...)
db.record_change(...)

# Jira
jira = JiraClient()
issue_data = jira.get_issue_as_requirement("JIRA-123")

# Files (directly or via services)
import os
with open(path, 'w') as f:
    f.write(content)
```

### 4. LLM Calls

Pattern for OpenAI (main agents):

```python
from openai import OpenAI
client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

response = client.chat.completions.create(
    model="gpt-4-turbo",
    messages=[{"role": "user", "content": prompt}],
    temperature=0.7,
    max_tokens=2000
)
result = response.choices[0].message.content.strip()
```

Pattern for Groq (evaluators):

```python
from groq import Groq
client = Groq(api_key=os.getenv("GROQ_API_KEY"))

response = client.chat.completions.create(
    model="mixtral-8x7b-32768",
    messages=[{"role": "user", "content": prompt}],
    temperature=0.3,  # Lower for consistency
    max_tokens=500
)
result = response.choices[0].message.content.strip()
```

### 5. Type Hints

All functions must have type hints:

```python
def process(self, state: RunState) -> RunState:
    pass

def fetch_requirement(self, key: str) -> Dict[str, Any]:
    pass

def analyze_coverage(
    self,
    requirements: List[str],
    features: Dict[str, str]
) -> Dict[str, Any]:
    pass
```

### 6. Error Handling

Log errors, don't silently fail:

```python
try:
    # operation
except Exception as e:
    self.log_error(
        "Operation failed",
        error=e,
        run_id=state.run_id,
        requirement_id=state.requirement_id
    )
    state.errors.append(f"Operation failed: {str(e)}")
    return state
```

## State Management

RunState is immutable for LangGraph:

```python
# ✅ CORRECT: Return new/updated state
state.status = "new_status"
state.coverage_markdown_path = path
return state

# Don't modify in-place and forget to return
```

## Database Schema (SQLite via MCP)

Tables managed via `services/db.py`:

- `requirements`: Requirement master records
- `runs`: Pipeline execution records
- `artifacts`: Generated files (features, coverage, tests)
- `locks`: Prevent concurrent modification
- `changes`: Change history with semantic vs wording-only classification

## Common Tasks

### Adding a New Agent

1. Create `agents/new_agent.py` with class extending LoggingMixin
2. Implement `process(state: RunState) -> RunState` method
3. Add `def new_agent_node(state: RunState) -> RunState:` function
4. Add node to `graph.py`: `graph.add_node("name", new_agent_node)`
5. Add edges in graph to connect to pipeline
6. Update router or conditional edges if needed
7. Add import to `agents/__init__.py`

### Modifying LLM Prompts

All prompts in agent files. Key agents:

- `classification_agent.py` - area classification (line ~40)
- `feature_gen_agent.py` - Gherkin generation (line ~60)
- `coverage_eval_agent.py` - coverage assessment (line ~60)
- `playwright_agent.py` - step def + page object (line ~115)
- `post_eval_agent.py` - final QA (line ~60)

### Adding Manual Test Documentation

Extend coverage evaluation prompt in `coverage_eval_agent.py` to detect and document manual tests.

### Testing Locally

```bash
# Activate venv
source venv/bin/activate

# Run specific requirement
python main.py run --text "test requirement"

# Verbose mode
python main.py run --jira-key PROJ-1 --verbose

# Initialize system
python main.py init
```

## Troubleshooting Patterns

### Debug LLM Output

```python
# Print raw LLM response before processing
self.log_debug(f"Raw response: {response}")

# Enable LANGSMITH to trace all calls
# Set LANGSMITH_ENABLED=true in .env
```

### Database Issues

```bash
# Check database
sqlite3 db/context.db "SELECT * FROM requirements LIMIT 5;"

# Reset database
rm db/context.db
python main.py init
```

### Import Errors

Check `services/logging_utils.py` - ensure setup_logging() called early.

## Performance Considerations

- OpenAI calls: ~2-5 seconds each
- Groq calls: ~1-3 seconds each
- File I/O: < 100ms
- SQLite queries: < 50ms
- Total pipeline: ~30-60 seconds per requirement

## Testing Strategy

- Unit tests in `tests/` (not yet implemented)
- Integration test: `python main.py run --text "..."`
- Manual QA: Review generated files in `playwright/ui/features/`

## Documentation

- `README.md` - System design & architecture
- `README-usage.md` - CLI usage & troubleshooting
- `coverage/example-coverage.md` - Sample output

## Common Mistakes to Avoid

❌ **Direct sqlite3 calls** → Use `services/db.py`  
❌ **Direct file operations without MCP** → Use abstraction  
❌ **Unhandled LLM errors** → Always try/except and log  
❌ **State mutations without return** → Always return state  
❌ **Missing type hints** → Add for all functions  
❌ **Silent failures** → Always log errors  
❌ **Hardcoded API keys** → Use .env files  
❌ **Multi-word prompts without temperature tuning** → Use 0.7 for gen, 0.3 for eval  

## Asking Copilot for Help

Good prompts:

> "Add a new agent for API test generation following the LoggingMixin pattern. It should analyze OpenAPI specs and generate pytest tests using similar structure to feature_gen_agent."

> "The classification agent sometimes misclassifies as 'other'. Improve the prompt to better distinguish between frontend/api/data/load/security."

> "Add LangSmith tracing to all LLM calls and verify spans are created for each agent."

## Contact & Resources

- LangGraph docs: https://langchain-ai.github.io/langgraph/
- OpenAI API: https://platform.openai.com/docs/api-reference
- Groq API: https://console.groq.com/docs
- Playwright: https://playwright.dev
- Pydantic: https://docs.pydantic.dev

---

**Last Updated:** 2025-11-16  
**Maintained By:** QAe2e Team

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wonderfull) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
