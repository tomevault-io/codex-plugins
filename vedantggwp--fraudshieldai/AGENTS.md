# FraudShield AI - Coding Instructions

## Architecture Overview

FraudShield is a **hybrid fraud detection system** combining rule-based anomaly detection with AI-powered explanations. The system uses a **provider abstraction pattern** to enable development without API keys and production flexibility.

**Key architectural layers:**
- **Frontend**: Next.js 14 (App Router) → communicates via REST to backend
- **API Layer**: FastAPI with Pydantic models → orchestrates services
- **Service Layer**: `AnomalyDetector` (scoring) + `ExplanationGenerator` (LLM explanations)
- **Provider Layer**: Abstract interfaces (`LLMProvider`, `PatternMatcher`) with pluggable implementations

## Critical Patterns

### 1. Provider Abstraction (Core Design Pattern)

All external services use **Protocol-based abstractions** to swap implementations:

```python
# Services define Protocol interfaces (app/services/)
class AnomalyDetectorProtocol(Protocol):
    def calculate_risk_score(self, transaction: dict) -> tuple[float, list[str]]: ...

# Providers implement abstract base classes (app/providers/)
class LLMProvider(ABC):
    @abstractmethod
    async def generate_explanation(self, request: ExplanationRequest) -> ExplanationResponse: ...
```

**Implementation locations:**
- `app/providers/llm/` → `MockLLMProvider`, `AzureOpenAIProvider` (future)
- `app/providers/patterns/` → `LocalJSONPatternMatcher`, `AzureSearchProvider` (future)

**Why:** Enables offline development with mock providers, zero vendor lock-in, and seamless production deployment.

### 2. Risk Scoring Methodology

Scoring is **deterministic and weighted** (not ML-based). See `METHODOLOGY.md` for full algorithm.

**Implementation:** `app/services/anomaly_detector.py` → `MockAnomalyDetector.calculate_risk_score()`

**Key factors** (defined in `app/config.py`):
- `NEW_PAYEE` (0.25) → first transaction to payee
- `UNUSUAL_TIMING` (0.25) → outside 9am-6pm
- `AMOUNT_SPIKE` (0.30) → >3× average (£520 baseline)
- `SUSPICIOUS_REFERENCE` (0.15) → contains "URGENT", "ASAP", etc.

**Risk thresholds:**
- `≥0.65` = HIGH (red)
- `0.35-0.64` = MEDIUM (amber)
- `<0.35` = LOW (green)

**Always cap scores at 1.0:** `min(score, 1.0)`

### 3. Data Flow: Transaction Creation

1. **POST /transactions** → `app/main.py:create_transaction()`
2. Calculate risk → `detector.calculate_risk_score(transaction_data)` returns `(score, factors)`
3. Map to risk level → `get_risk_level(score)` applies thresholds
4. Store → `transaction_store.add()` (in-memory dict in `app/storage.py`)
5. Return → `TransactionResponse` model (no explanation yet)

**Explanation generation is lazy** (only on GET `/transactions/{id}`):
- Calls `explanation_gen.generate_explanation()` 
- Uses `MockExplanationGenerator` by default (template-based, no API calls)

### 4. Frontend Data Fetching

**Always use SWR for data fetching** (see `frontend/hooks/`):

```typescript
// Pattern for listing
const { transactions, stats, isLoading } = useTransactions();

// Pattern for detail
const { transaction, isLoading, isError } = useTransaction(id);
```

**API client location:** `frontend/lib/api.ts` → wraps fetch with error handling
**Base URL:** `process.env.NEXT_PUBLIC_API_URL` (defaults to `http://localhost:8000`)

### 5. Component Structure

**Page components** (`frontend/app/`) → use client hooks for data
**UI components** (`frontend/components/`) → organized by domain:
- `dashboard/` → transaction table, stats row
- `detail/` → risk badge, confidence meter, factor cards
- `layout/` → header, theme toggle
- `ui/` → shadcn-style base components (badge, button, card)

**Styling:** Tailwind CSS with dark mode via `next-themes` (see `frontend/components/layout/providers.tsx`)

## Development Workflows

### Running Locally

```bash
# Backend (Terminal 1)
source venv/bin/activate
uvicorn app.main:app --reload --port 8000

# Frontend (Terminal 2)
cd frontend && npm run dev
```

**Seed data:** Automatically loaded on startup from `app/data/demo_transactions.json` (see `app/main.py:lifespan()`)

### Deployment Architecture

**Production Stack:**
- **Backend:** Render.com (FastAPI with Gunicorn + Uvicorn workers)
  - Config: `render.yaml` + `Procfile`
  - Command: `gunicorn --worker-class uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000 -w 2 --timeout 120 app.main:app`
  - Health check: `/health` endpoint
- **Frontend:** Vercel (Next.js 14)
  - Environment: Set `NEXT_PUBLIC_API_URL` to Render backend URL
  - Auto-deploys from `main` branch

**Critical:** Frontend's `NEXT_PUBLIC_API_URL` must point to production backend, not localhost

### Provider Configuration

Controlled via environment variables (defaults work without keys):

```bash
LLM_PROVIDER=mock           # mock | azure_openai | openai | ollama
PATTERN_PROVIDER=local_json # local_json | azure_search | mock
```

**Default (mock):** No external calls, deterministic responses, perfect for development/testing.

### Adding New Risk Factors

1. Add weight to `app/config.py:SCORING_WEIGHTS`
2. Implement detection logic in `app/services/anomaly_detector.py:calculate_risk_score()`
3. Add template to `app/providers/llm/mock_provider.py:FACTOR_TEMPLATES`
4. Update `METHODOLOGY.md` with factor definition

## Integration Points

- **FastAPI ↔ Frontend:** REST API with Pydantic validation (see `app/models.py`)
- **Service ↔ Providers:** Dependency injection via `Depends()` (FastAPI pattern)
- **Storage:** In-memory dict (`app/storage.py:TransactionStore`) → replace with DB for production

## Common Mistakes to Avoid

- **Don't bypass provider abstractions** → always use Protocol/ABC interfaces
- **Don't hardcode risk thresholds** → use `RISK_THRESHOLDS` from `app/config.py`
- **Don't call explanations on POST** → it's expensive, only generate on GET detail view
- **Don't forget CORS** → middleware already configured in `app/main.py`
- **Don't use `cache: 'force-cache'`** in frontend → use `cache: 'no-store'` for real-time fraud data
- **Don't use in-memory storage for production** → current `TransactionStore` loses data on restart (replace with database)

## Current Production Gaps

**Blocking Issues:**
1. **No persistence** - In-memory storage loses data on Render dyno restart
2. **No authentication** - API is completely open (anyone can access)
3. **No transaction creation UI** - Users must use API directly
4. **Frontend env mismatch** - Likely pointing to localhost instead of Render

See [PRODUCTION_READINESS.md](../PRODUCTION_READINESS.md) for complete assessment and roadmap.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vedantggwp)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/vedantggwp)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
