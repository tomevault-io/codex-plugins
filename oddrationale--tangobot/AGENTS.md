# AGENTS.md

## Project Overview
TangoBot is a Python monorepo for Tango Charities internal tooling. It combines ML photo analysis (face detection + age estimation), API services, a GroupMe bot, a LangGraph agent, a CLI, and a HuggingFace Space UI. Development is centered on a `uv` workspace, with `Aspire 13` orchestrating multi-service local runs.

## Tech Stack & Versions
- Python: `>=3.12` (workspace-wide)
- Package manager / workspace: `uv` (`pyproject.toml` at repo root, shared `uv.lock`)
- Service frameworks: FastAPI, Uvicorn, LangGraph, Gradio
- Orchestration: `Aspire 13` (`apphost.cs`)
- Test framework: `pytest` (+ `pytest-asyncio`, `pytest-cov`, `pytest-httpx`)
- Build backend: Primarily `uv-build` and Hatchling for others

## Quickstart
Start full stack (recommended):
   - `aspire run`

## Repo Map: Where To Change What
- `packages/face-age-inference/`: core ML inference engine/config/image utilities.
- `packages/groupme-client/`: GroupMe HTTP client.
- `packages/common/`: shared Pydantic schemas/utilities across apps.
- `apps/ml-api/`: FastAPI inference wrapper service.
- `apps/groupme-bot/`: GroupMe webhook handler.
- `apps/tangobot-agent/`: LangGraph/CopilotKit-based agent + tools.
- `apps/rag-pipeline/`: batch PDF ingestion into PG vector store.
- `apps/huggingface-space/`: Gradio UI app for HF Space deployment.
- `apps/cli/`: `tangobot-cli` CLI entrypoint.
- `apphost.cs`: Aspire orchestration and service wiring.
- `.github/workflows/sync-to-hf-spaces.yml`: CI deployment workflow to HuggingFace Space.
- `README.md`, app/package `README.md`: usage docs.

## Testing & CI Expectations
- None yet

## Git & PR Workflow
- Use small, focused commits and include test evidence in PR description.
- Important: pushes to `main` that touch HF deployment paths trigger automatic HuggingFace deployment.

## Security & Secrets
- Never commit secrets from local/dev config:
  - `appsettings.Development.json` (gitignored)
  - `.env` / `.env.*` files
  - API keys/tokens (OpenAI, LangSmith, GroupMe, HuggingFace)
- Treat these as sensitive environment inputs:
  - `OPENAI_API_KEY`, `LANGSMITH_*`, `GROUPME_*`, DB credentials, blob connection strings.
- Do not paste secrets in issues/PR comments/log snippets.

### ⚠️ Ask first
- Large dependency upgrades (LangChain/FastAPI/CopilotKit/OpenTelemetry).
- Changes to deployment workflow or Aspire orchestration.
- Any operation that needs networked credentials or production-like data.

### 🚫 Never do
- Commit secrets, `.env` files, or `appsettings.Development.json`.
- Edit generated/cache artifacts (`.venv/`, `__pycache__/`, `.pytest_cache/`, coverage outputs) as source changes.

## Python Style

> Write code. Not too much. Mostly functions.
> Make illegal states unrepresentable.
> Functional core, imperative shell.

### Defaults

- Standalone functions over classes. A class must justify its existence (lifecycle, invariant, protocol).
- `@dataclass(frozen=True, slots=True)` for domain objects. Immutable by default.
- `Enum` for closed sets. Never raw strings for status/kind/mode.
- Union types (`A | B | C`) for mutually exclusive states. Not optional fields on one blob.
- `T | None` over `Optional[T]`.
- ~15 lines per function, ~150 lines per module. Split when exceeded.
- No inheritance. Compose.

### Illegal states

Validate at boundaries (handlers, CLI, file I/O). Once data enters the core, it is correct by construction.

```python
# ✗ Stringly-typed
def create_user(role: str): ...

# ✓ Type-driven
class Role(Enum):
    ADMIN = "admin"
    VIEWER = "viewer"

@dataclass(frozen=True, slots=True)
class User:
    name: str
    email: Email  # validated wrapper type
    role: Role
```

```python
# ✗ Optional field soup — which fields are None when?
@dataclass
class Payment:
    status: str
    paid_at: datetime | None = None
    error: str | None = None

# ✓ Each state carries exactly its data
@dataclass(frozen=True)
class Pending: ...
@dataclass(frozen=True)
class Paid:
    paid_at: datetime
@dataclass(frozen=True)
class Failed:
    error: str
type Payment = Pending | Paid | Failed
```

### Functional core, imperative shell

Core functions: take values, return values. No `await`, no I/O, no `datetime.now()`, no env vars. Pass these in as arguments.

Shell functions: thin wiring only — fetch → call core → persist.

```python
# core (pure, testable with plain assert)
def calculate_discount(order: Order, rules: list[Rule]) -> Decimal: ...
def build_invoice(order: Order, discount: Decimal) -> Invoice: ...

# shell (I/O, wiring)
async def handle_checkout(order_id: str, db: DB) -> None:
    order = await db.fetch_order(order_id)
    rules = await db.fetch_rules()
    discount = calculate_discount(order, rules)
    invoice = build_invoice(order, discount)
    await db.save(invoice)
```

### Avoid

- Classes with one method (use a function)
- Mutable dicts as domain objects
- Raising exceptions for control flow (return values)
- Mixed I/O and logic in the same function
- Code "for later" — solve the stated problem only

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/oddrationale)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/oddrationale)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
