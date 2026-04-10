# Copilot Instructions for Para-Zero

Para-Zero is a **Dynamic Application Security Testing Research Framework** that automates vulnerability verification through AI-generated code. This document guides AI agents to be immediately productive.

## Architecture Overview

**Three-Layer Pipeline:**
1. **Intelligence (pulse_monitor.py):** Aggregates CVE data from NVD/RSS feeds → `data/research_queue/`
2. **Synthesis (core/template_gen.py):** LLM generates verification code → validates with AST firewall → deploys to `modules/`
3. **Execution (engine.py):** Runs verification modules on test traffic; logs findings to `findings.json`

**Critical Data Flow:**
```
CVE JSON (research_queue/)
  ↓ [TemplateSynthesizer reads]
  ↓ [LLMClient generates code]
  ↓ [ASTValidator firewall checks]
  ├→ SAFE: modules/cve_XXXX.py [hot-loaded by engine.py]
  └→ UNSAFE: data/quarantine/ [rejected]
```

## Multi-Provider LLM Architecture (NEW)

Para-Zero now supports **local-first (Ollama) + cloud fallback** LLM providers:

**Provider Hierarchy:**
- Primary: **Ollama** (local, OpenAI-compatible, `http://localhost:11434/v1`)
- Fallback: **OpenAI** (GPT-4), **Anthropic** (Claude)

**Task-Aware Model Selection:**
- `task_type="reasoning"` → Uses `MODEL_REASONING` (default: llama3)
- `task_type="coding"` → Uses `MODEL_CODING` (default: deepseek-coder-v2)

**Configuration:** All settings in `config.py` via environment variables:
```python
LLM_PROVIDER = os.getenv("LLM_PROVIDER", "ollama")
OLLAMA_BASE_URL = "http://localhost:11434/v1"
MODEL_REASONING = "llama3"
MODEL_CODING = "deepseek-coder-v2"
```

**Academy Mode (New):** `train.py` ingests CTF writeups and local knowledge:
```bash
python train.py --url "https://ctf-writeup.com" --tags "RCE" "Spring"
python train.py --file research.md --tags "CVE-2024-1234"
```
Extracts content → sanitizes to 6,000 words → saves to `data/research_queue/` with ACADEMY_MODE tag.

## Code Generation & Safety Pipeline

### System Prompts (Task-Specific)

**Reasoning Prompt** (for CTF/knowledge analysis):
- Extracts key vulnerability concepts
- Summarizes exploit techniques educationally
- Focuses on defensive lessons

**Coding Prompt** (for verification module synthesis):
```
"Inherit from BaseVerifier. Implement verify(target_url, session).
Return True if vulnerable, False otherwise. Non-destructive only.
FORBIDDEN: os, sys, subprocess, eval, exec, open(write), while True."
```

### ASTValidator Firewall (core/ast_validator.py)

**Enforced Constraints:**
| Rule | Forbidden | Reason |
|------|-----------|--------|
| Imports | os, sys, subprocess, shutil, socket, pickle | System/network access |
| Functions | eval(), exec(), __import__() | Code injection |
| File I/O | open() with write/append modes | Data modification |
| Loops | while True without break | DoS |
| Interface | Must inherit from BaseVerifier | Contract enforcement |
| Methods | Must implement verify() | Execution interface |

**Workflow:**
```python
from core.ast_validator import validate_generated_code
is_safe, violations = validate_generated_code(code_string)
if not is_safe:
    # violations = list of security rule violations
    # Save to data/quarantine/
```

### BaseVerifier Interface (core/base_verifier.py)

All generated modules MUST implement:
```python
from core.base_verifier import BaseVerifier
import requests

class Verifier_CVE_2024_1234(BaseVerifier):
    def verify(self, target_url: str, session: requests.Session) -> bool:
        """Return True if target_url is vulnerable."""
        # Only read-only operations
        response = session.get(target_url, timeout=5)
        return "vulnerable_indicator" in response.text
```

## Directory Structure & Key Patterns

**Input Queues:**
- `data/research_queue/` → JSON files from NVD connector or Academy Mode
- Format: `{"id": "CVE-2024-1234", "description": "...", "cvss_score": 9.8}`

**Processing Pipeline:**
```
core/
  ├── base_verifier.py          # Abstract interface all code inherits from
  ├── ast_validator.py          # Firewall: validate_generated_code()
  ├── llm_client.py             # Multi-provider: LLMClient, create_llm_client()
  ├── template_gen.py           # Main loop: TemplateSynthesizer.process_queue()
  ├── module_loader.py          # Hot-loader: dynamically imports modules/
  ├── scope.py                  # Authorization: ScopeMarshall checks allowlist
  └── train.py                  # Academy Mode: SmartScraper ingests content
```

**Output Artifacts:**
- `modules/cve_XXXX_YYYY.py` → Approved verification code (hot-loaded)
- `data/archive/` → Processed CVE JSON files
- `data/quarantine/` → Rejected code + violation reports
- `findings.json` → Engine output: identified vulnerabilities
- `logs/` → Per-component audit trails (template_gen.log, engine.log, ast_validator.log, etc.)

## Developer Workflows

### Adding New Verification Module

1. **Write CVE JSON to `data/research_queue/`:**
   ```json
   {
       "id": "CVE-2024-XXXX",
       "description": "Detailed vulnerability description...",
       "cvss_score": 9.8
   }
   ```

2. **TemplateSynthesizer auto-processes:**
   - LLMClient generates verification code
   - ASTValidator validates
   - If safe → moves to `modules/`, if unsafe → quarantine

3. **Engine hot-loads new module** (automatic via ModuleLoader)

### Testing Generated Code

```python
# Test locally before deploying
from core.base_verifier import BaseVerifier
from core.ast_validator import validate_generated_code
from modules import cve_2024_1234

# Validate AST
is_safe, violations = validate_generated_code(open("modules/cve_2024_1234.py").read())
assert is_safe, f"Code unsafe: {violations}"

# Run verification
import requests
verifier = cve_2024_1234.Verifier_CVE_2024_1234()
result = verifier.verify("https://target.com", requests.Session())
```

### Running the Full Pipeline

```bash
# Terminal 1: Start Ollama (if using local)
ollama serve

# Terminal 2: Pull required models
python setup/pull_models.py

# Terminal 3: Run command center (orchestrates all processes)
python main.py
# Starts: NVD connector, template synthesizer, verification engine, dashboard
```

## LLMClient Usage Patterns

### Pattern 1: Generate Verifier Code (Legacy)
```python
from core.llm_client import LLMClient

client = LLMClient()  # Auto-detects provider from config
code = client.generate_verifier_code(
    cve_id="CVE-2024-1234",
    description="Apache 2.4.49 path traversal...",
    cvss_score=9.8
)
# Returns Python code ready to execute
```

### Pattern 2: Task-Aware Generation (New)
```python
# Reasoning task (uses MODEL_REASONING)
analysis = client.generate(
    prompt="Analyze this CTF writeup for defensive lessons...",
    task_type="reasoning",
    max_tokens=2048,
    temperature=0.3
)

# Coding task (uses MODEL_CODING)
code = client.generate(
    prompt="Generate verification for CVE-2024-1234...",
    task_type="coding",
    max_tokens=1500,
    temperature=0.1
)
```

### Pattern 3: Provider Override
```python
# Use OpenAI instead of default Ollama
client = LLMClient(provider="openai", api_key="sk-...")
result = client.generate(prompt, task_type="coding")

# Factory function with error handling
from core.llm_client import create_llm_client
client = create_llm_client()  # Returns None if all providers fail
```

## Authorization & Scope (scope.py)

**Critical:** Para-Zero only verifies on **whitelisted domains**.

```python
from core.scope import ScopeMarshall

# Check if domain is authorized
is_authorized = ScopeMarshall.check_authorization("example.com")
if not is_authorized:
    # Log denial, skip verification
```

**Config file:** `scope.allowlist.json`
```json
{
    "authorized_domains": ["staging.example.com", "qa.example.com"],
    "test_accounts": {"primary": "user1", "secondary": "user2"}
}
```

## Logging Pattern

Every component logs to `logs/{component_name}.log`:
```python
import logging

logger = logging.getLogger("template_gen")
handler = logging.FileHandler("logs/template_gen.log")
formatter = logging.Formatter("%(asctime)s - %(name)s - %(levelname)s - %(message)s")
handler.setFormatter(formatter)
logger.addHandler(handler)

logger.info("Processing CVE-2024-1234")
logger.error("AST validation failed: forbidden import detected")
```

## Configuration via Environment Variables

All settings in `.env` (loaded by `config.py`):
```bash
# LLM Configuration
LLM_PROVIDER=ollama                          # "ollama", "openai", "anthropic"
OLLAMA_BASE_URL=http://localhost:11434/v1   # For Ollama
MODEL_REASONING=llama3                       # Analysis model
MODEL_CODING=deepseek-coder-v2               # Code generation model
OPENAI_API_KEY=sk-...                        # If using OpenAI
ANTHROPIC_API_KEY=sk-ant-...                 # If using Anthropic

# API Configuration
NVD_API_KEY=your_key                         # For NVD connector
LOG_LEVEL=INFO                               # DEBUG, INFO, WARNING, ERROR

# Academy Mode
ACADEMY_MODE_ENABLED=true                    # Enable smart scraper
```

## Important Constraints

1. **No Cloud Defaults:** Ollama (local) is default provider → zero external API exposure by default
2. **Non-Destructive Only:** Generated code must never modify files, system state, network
3. **BaseVerifier Contract:** ALL generated classes must inherit and implement interface
4. **6000-Word Limit:** Academy Mode sanitizes content to respect model context windows
5. **No Markdown in Generated Code:** System prompts enforce pure Python output
6. **Graceful Degradation:** LLMClient returns `None` on failure, never raises exceptions

## Common Patterns to Follow

- **Always use `Optional[str]`** return types for LLM methods (can fail)
- **Logging before critical operations** (especially in generation/validation pipeline)
- **Configuration via environment** (never hardcode API keys or provider names)
- **Quarantine unsafe code immediately** (don't try to fix, just reject)
- **Test Authorization before verification** (ScopeMarshall gate all engine operations)
- **Task-specific prompts** (reasoning vs coding - don't mix)

## Files to Read First

1. **SYNTHESIS_CORE.md** → Code generation architecture & security guarantees
2. **ORCHESTRATION_UI.md** → Multi-process orchestration & hot-loading
3. **LOCAL_FIRST_MIGRATION.md** → Multi-provider LLM setup
4. **core/template_gen.py** → Main synthesis loop (entry point: `TemplateSynthesizer.process_queue()`)
5. **core/ast_validator.py** → Security firewall (defines forbidden patterns)

---

**Status:** Production-Ready | **Last Updated:** January 27, 2026

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SecScholar)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/SecScholar)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
