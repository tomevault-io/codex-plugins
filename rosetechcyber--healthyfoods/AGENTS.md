# Copilot instructions for HealthyFoods

## Current repository reality
- This repository contains a **full FastAPI application**: `HealthyFoods` — an AI Agentic Food Ordering Platform.
- The application is production-ready with Azure integration, CI/CD pipeline, Docker support, and a complete test suite.
- Stack: **Python 3.11+**, **FastAPI**, **Pydantic v2**, **Azure OpenAI / Content Safety / Storage / Service Bus**, **Docker**.

## Source of truth
- Product direction: `README.md`.
- API reference: `API_DOCUMENTATION.md`.
- Legal constraints: `LICENSE`.
- Deployment details: `DEPLOYMENT.md`.
- Security posture: `SECURITY.md`, `SECURITY_SUMMARY.md`.

## Existing agent rule files
- This file (`.github/copilot-instructions.md`) is the only agent instruction file.

## Application structure
```
app/
  main.py               # FastAPI app entry-point, lifespan, routers, health check
  core/
    config.py           # Pydantic-settings (SECRET_KEY required via env)
    security.py         # Middleware, hashing, webhook signature verification
  api/
    orders.py           # POST/GET /api/v1/orders
    customers.py        # POST/GET /api/v1/customers
    catering_firms.py   # POST/GET /api/v1/catering-firms
    payments.py         # POST/GET /api/v1/payments
    delivery.py         # POST/GET /api/v1/delivery
  models/
    schemas.py          # Pydantic v2 request/response schemas
  services/
    ai_agent.py         # AIAgentOrchestrator (Azure OpenAI-backed)
    azure_services.py   # Azure Storage, Service Bus, Content Safety clients
tests/
  conftest.py           # Fixtures; sets SECRET_KEY=test-secret-key-for-ci
  test_main.py          # Root / health endpoint tests
  test_orders.py        # Order creation & validation tests
  test_ai_agent.py      # AI agent logic tests
  test_security.py      # Security module tests
```

## How to run locally
```bash
cp .env.example .env          # fill in your Azure credentials
pip install -r requirements.txt
python -m uvicorn app.main:app --reload
# API docs: http://localhost:8000/docs
```

## How to test
```bash
SECRET_KEY='test-secret-key-for-ci' pytest tests/ --cov=app
```

## How to build & run with Docker
```bash
docker build -t healthyfoods .
docker run -p 8000:8000 -e SECRET_KEY='your-secret-key' healthyfoods
```

## How to make changes in this codebase
- Follow the existing module boundaries: API layer → service layer → Azure integrations.
- Keep naming aligned with repository identity (`HealthyFoods`).
- `SECRET_KEY` is a **required** environment variable — never hard-code it; always supply via env or secrets.
- All new endpoints must have corresponding tests in `tests/`.
- Update `API_DOCUMENTATION.md` when changing public API contracts.

## Architecture guidance for future edits
- Maintain a clear separation between:
  - user-facing ordering experience (`app/api/`)
  - AI decision/mediation logic (`app/services/ai_agent.py`)
  - operational integrations (`app/services/azure_services.py`)
- Prefer explicit interface boundaries between these areas.
- Record integration points (provider name, auth method, request/response contracts) in `API_DOCUMENTATION.md` as they are added.

## What to avoid
- Do not add generic "best practice" docs disconnected from actual code.
- Do not rename the repository or rewrite product scope without explicit user request.
- Do not introduce multiple frameworks or polyglot stacks in one pass.
- Do not commit real secrets — use `.env` locally and Azure Key Vault / GitHub Secrets in production.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RoseTechCyber) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
