# 🤖 COPILOT AUTO-UPDATE RULE

**Copilot MUST update this file automatically when ANY of the following happens:**

1. **User defines or changes** project domain, stack, database, or scale → Update 🎯 PROJECT IDENTITY
2. **User starts a new task** or completes one → Update 🚧 CURRENT FOCUS and ✅ COMPLETED WORK
3. **User makes architectural decisions** → Update 📋 IMPORTANT CONTEXT and 🏗 PATTERNS TO USE
4. **User adds explicit instructions** (e.g., "always do X", "use Y for Z") → Add to 📜 USER INSTRUCTIONS LOG
5. **User provides credentials or config names** → Add NAME ONLY to 🔐 CREDENTIALS & CONFIG (⚠️ NEVER store values!)
6. **User says "don't do X"** or prohibits something → Add to 🚫 USER SAID "DON'T DO THIS"
7. **User shares important context** (business rules, constraints, domain knowledge) → Add to 📋 IMPORTANT CONTEXT

**After updating, briefly confirm what was changed at the end of the response.**

---

## 🎯 PROJECT IDENTITY

| Field        | Value                                                                                                  |
| ------------ | ------------------------------------------------------------------------------------------------------ |
| **Name**     | LLM Verify                                                                                             |
| **Domain**   | AI model verification & benchmarking — detect model fraud (e.g., resold APIs misrepresenting identity) |
| **Stack**    | Python 3.12+ · FastAPI · Pydantic v2 · httpx (async) · SQLAlchemy 2.0 (async) · Alembic                |
| **Database** | SQLite (dev & prod — file-based, zero-config)                                                          |
| **Scale**    | Single-node CLI + web dashboard · benchmarks run locally or via CI                                     |
| **Repo**     | `benchmark/`                                                                                           |

---

## 📜 USER INSTRUCTIONS LOG

| #   | Date       | Instruction                                      |
| --- | ---------- | ------------------------------------------------ |
| 1   | 2026-02-17 | Project bootstrapped with Copilot context system |
|     |            |                                                  |

---

## ✅ COMPLETED WORK

| #   | Date       | Task                                                                    |
| --- | ---------- | ----------------------------------------------------------------------- |
| 1   | 2026-02-17 | Project bootstrap — copilot context, settings, gitignore                |
| 2   | 2026-02-17 | Full project scaffolding — 30+ files, all layers, 32 prompts            |
| 3   | 2026-02-17 | All 9 unit tests passing                                                |
| 4   | 2026-02-17 | Renamed to LLM Verify, pushed to GitHub                                 |
| 5   | 2026-02-17 | Fixed factory: suspect provider now uses Anthropic protocol by default  |
| 6   | 2026-02-17 | First live benchmark — identity probes vs suspect API (opuscode.pro)    |
| 7   | 2026-02-17 | Confirmed fraud: suspect serves Claude 3.5 Sonnet as Claude Sonnet 4    |
| 8   | 2026-02-17 | Updated README with no-API-key usage guide and red flags doc            |
| 9   | 2026-02-17 | Added deep analysis feature — service, schemas, handler, README section |

---

## 🚧 CURRENT FOCUS

| Item           | Detail                                                              |
| -------------- | ------------------------------------------------------------------- |
| **Working on** | Deep analysis feature complete — ready for live testing             |
| **Blockers**   | None                                                                |
| **Next up**    | Live test deep analysis endpoint, web dashboard, more prompt suites |

---

## 🔐 CREDENTIALS & CONFIG

> ⚠️ **NEVER store actual values here — names/keys only!**

| #   | Name                 | Service      | Notes                                |
| --- | -------------------- | ------------ | ------------------------------------ |
| 1   | SUSPECT_API_KEY      | opuscode.pro | Suspect API key — Anthropic protocol |
| 2   | SUSPECT_API_BASE_URL | opuscode.pro | https://opuscode.pro/api             |

---

## 🚫 USER SAID "DON'T DO THIS"

| #   | Date | Prohibition |
| --- | ---- | ----------- |
|     |      |             |

---

## 📋 IMPORTANT CONTEXT

- **Core Problem:** Users are being sold API access to models misrepresented as premium models (e.g., Kimi sold as Claude). The system prompt says "Claude" but the underlying model is actually Kimi.
- **Goal:** Build a benchmark suite that can fingerprint AI model behavior to verify true model identity, comparing response patterns, capabilities, and quirks across models.
- **Suspect API (opuscode.pro):** Uses **Anthropic Messages protocol**, NOT OpenAI. Endpoint: `https://opuscode.pro/api/v1/messages`. Auth header: `x-api-key`. Available models: `Opus 4.6`, `Sonnet 4.5`, `Haiku 4.5` (their naming). Default model: `Opus 4.6`.
- **First test result:** Suspect claims to be Claude Sonnet 4 but self-identifies as **claude-3-5-sonnet-20241022** (Claude 3.5 Sonnet). Gave 3 different knowledge cutoffs, mentions "custom proxy server", avg latency 14s.
- **Factory mapping:** `suspect` provider defaults to `anthropic` protocol. Can be overridden via `protocol` field in ModelConfig.
- **Key Features Planned:**
  - Run standardized prompt suites against multiple API endpoints
  - Collect and store structured benchmark results (latency, token usage, response quality)
  - Statistical comparison & fingerprinting to detect model identity
  - Web dashboard to visualize results
  - CLI for running benchmarks in CI/CD

---

## 🚨 HARD RULES

### Security

- ❌ **NEVER** commit secrets, API keys, or tokens to code or config files
- ✅ Use `.env` files (gitignored) and `pydantic-settings` for all secrets
- ✅ Parameterized queries only — no string interpolation in SQL
- ✅ Validate all external input with Pydantic models

### Performance

- ✅ Use `async/await` for all I/O (HTTP calls, DB queries, file ops)
- ✅ Use `httpx.AsyncClient` with connection pooling for API calls
- ✅ Use SQLAlchemy async sessions with proper context managers
- ✅ Batch concurrent API calls with `asyncio.gather()` where appropriate

### Architecture

- ✅ Dependency injection via FastAPI `Depends()`
- ✅ Strict separation: handlers → services → repositories → models
- ✅ Each layer has a single responsibility
- ✅ Config is centralized in one place (`src/config.py`)

---

## 📐 CODE STYLE

- **Type hints** on ALL function signatures and return types
- **Docstrings** on all public functions (Google style)
- **Descriptive names** — no single-letter variables except `i`, `_` in comprehensions
- **Early returns** to reduce nesting
- **Max 30 lines** per function — extract helpers if longer
- **Pydantic models** for all data structures crossing boundaries
- **f-strings** for string formatting
- **`pathlib.Path`** over `os.path`

---

## 🏗 PATTERNS TO USE

| Pattern                | Usage                                                            |
| ---------------------- | ---------------------------------------------------------------- |
| **Result pattern**     | Return `Result[T, Error]` for operations that can fail           |
| **Service pattern**    | Business logic lives in service classes, not in handlers         |
| **Repository pattern** | DB access abstracted behind repository interfaces                |
| **Adapter pattern**    | Each AI provider gets an adapter implementing a common interface |
| **Factory pattern**    | Create model adapters dynamically from config                    |
| **Strategy pattern**   | Benchmark suites are pluggable strategies                        |

---

## 🚫 PATTERNS TO AVOID

| Anti-pattern              | Why                                                 |
| ------------------------- | --------------------------------------------------- |
| **God objects**           | Split into focused, single-responsibility classes   |
| **Magic numbers/strings** | Use enums and constants                             |
| **Mutable global state**  | Use DI and explicit passing                         |
| **Generic `utils.py`**    | Create specific modules (`string_helpers.py`, etc.) |
| **Bare `except:`**        | Always catch specific exceptions                    |
| **Print debugging**       | Use `structlog` or `logging`                        |
| **Nested callbacks**      | Use async/await                                     |

---

## 📁 PROJECT STRUCTURE

```
benchmark/
├── src/
│   ├── __init__.py
│   ├── main.py                  # FastAPI app entry point
│   ├── config.py                # Pydantic Settings configuration
│   ├── database.py              # SQLAlchemy engine & session setup
│   ├── handlers/                # API route handlers (thin layer)
│   │   ├── __init__.py
│   │   ├── benchmarks.py
│   │   └── results.py
│   ├── services/                # Business logic
│   │   ├── __init__.py
│   │   ├── benchmark_runner.py
│   │   ├── model_comparator.py
│   │   └── fingerprint.py
│   ├── repositories/            # Database access
│   │   ├── __init__.py
│   │   ├── benchmark_repo.py
│   │   └── result_repo.py
│   ├── models/                  # SQLAlchemy ORM models
│   │   ├── __init__.py
│   │   ├── benchmark.py
│   │   └── result.py
│   ├── schemas/                 # Pydantic request/response schemas
│   │   ├── __init__.py
│   │   ├── benchmark.py
│   │   └── result.py
│   ├── adapters/                # AI provider adapters
│   │   ├── __init__.py
│   │   ├── base.py              # Abstract base adapter
│   │   ├── openai_adapter.py
│   │   ├── anthropic_adapter.py
│   │   └── generic_adapter.py   # For OpenAI-compatible APIs
│   └── prompts/                 # Benchmark prompt suites
│       ├── __init__.py
│       ├── identity.py          # "Who are you?" probes
│       ├── capability.py        # Capability-specific tests
│       └── fingerprint.py       # Behavioral fingerprinting prompts
├── tests/
│   ├── __init__.py
│   ├── conftest.py              # Shared fixtures
│   ├── test_benchmark_runner.py
│   ├── test_model_comparator.py
│   └── test_adapters/
│       └── test_generic_adapter.py
├── alembic/                     # Database migrations
│   └── versions/
├── alembic.ini
├── .env.example
├── .gitignore
├── pyproject.toml
└── README.md
```

---

## 🔑 ENVIRONMENT VARIABLES

```env
# === REQUIRED ===
DATABASE_URL=sqlite+aiosqlite:///./benchmarker.db

# === AI PROVIDER API KEYS (add as needed) ===
# OPENAI_API_KEY=
# ANTHROPIC_API_KEY=
# SUSPECT_API_KEY=           # The API you're testing/verifying
# SUSPECT_API_BASE_URL=      # Base URL of the suspect API

# === OPTIONAL ===
# LOG_LEVEL=INFO
# BENCHMARK_TIMEOUT=30       # Seconds per API call
# MAX_CONCURRENT_CALLS=5     # Limit parallel API requests
```

---

## 📚 GLOSSARY

| Abbreviation     | Meaning                        |
| ---------------- | ------------------------------ |
| `ctx`            | Context                        |
| `repo`           | Repository                     |
| `svc`            | Service                        |
| `dto`            | Data Transfer Object           |
| `handler`        | API route handler (controller) |
| `adapter`        | AI provider adapter            |
| `cfg` / `config` | Configuration                  |
| `db`             | Database                       |
| `req` / `res`    | Request / Response             |
| `bench`          | Benchmark                      |
| `fp`             | Fingerprint                    |

---

## ✅ TESTING

- **Test alongside code** — tests mirror `src/` structure
- **Mock all externals** — API calls, database, file system
- **Cover edge cases** — empty inputs, timeouts, malformed responses
- **Target 80% coverage** minimum
- **Use `pytest`** with `pytest-asyncio` for async tests
- **Fixtures in `conftest.py`** — shared test data and mocks
- **Test naming:** `test_<function>_<scenario>_<expected>` (e.g., `test_run_benchmark_timeout_raises_error`)
- **Use `httpx.AsyncClient`** for integration testing FastAPI endpoints

---
> Source: [mintesnot-teshome/llm-verify](https://github.com/mintesnot-teshome/llm-verify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-07 -->
