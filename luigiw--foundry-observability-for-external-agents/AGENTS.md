# Copilot Instructions

## Architecture

This is a multi-cloud LangGraph customer support multi-agent system with a shared core and cloud-specific deployments:

```
shared/customer-support-agents/   ← Cloud-agnostic core (agents, graph, state)
aws/langgraph-customer-support/   ← AWS Lambda + Bedrock deployment
gcp/langgraph-customer-support/   ← GCP Cloud Run + Azure AI Foundry deployment
ui/                               ← Streamlit frontend
```

**Agent flow:** Router Agent (Haiku, fast/cheap) classifies queries → routes to one of four specialists (Billing, Technical, General, Escalation) using Sonnet.

**Key abstraction:** The shared package defines `BaseLLMProvider` and `CustomerSupportAgents` accepting an `llm_factory: Callable[[model_id, temperature], BaseLLMProvider]`. Cloud implementations inject their own LLM provider (AWS Bedrock or Azure AI Foundry via `ChatAnthropic`).

**State management:** `AgentState` is a TypedDict/Pydantic BaseModel using LangGraph's `Annotated[list, add_messages]` reducer for message accumulation. Routing uses `query_type`, `confidence`, `needs_escalation`, `handled_by`, and `final_response` fields.

**Tracing:** Both clouds send telemetry to Azure Application Insights using OpenTelemetry GenAI semantic conventions (`gen_ai.operation.name`, `gen_ai.provider.name`, etc.).

## Build & Test Commands

There is no unified build system. Each subdirectory is independent with its own virtualenv and `requirements.txt`.

### AWS (`aws/langgraph-customer-support/`)
```bash
pip install -r requirements.txt
python test_local.py          # runs 4 query-type tests (billing, technical, general, escalation)
sam build && sam deploy       # deploy to AWS
```

### GCP (`gcp/langgraph-customer-support/`)
```bash
pip install -r requirements.txt
uvicorn src.server:app --reload --port 8080   # local dev server
python test_local.py                           # agent tests
./test_local.sh                                # integration tests via curl
./deploy.sh                                    # deploy to Cloud Run
```

### UI (`ui/`)
```bash
pip install -r requirements.txt
streamlit run app.py
```

## Conventions

- **Shared-first development:** New agent logic goes in `shared/customer-support-agents/`. Cloud implementations only handle LLM instantiation, tracing, and HTTP/Lambda entry points.
- **LLM model aliases:** Use `"haiku"` and `"sonnet"` as model name abstractions in the shared layer. Cloud implementations map these to concrete model IDs.
- **Tracing pattern:** Wrap LLM calls with OpenTelemetry spans using `gen_ai.*` semantic convention attributes. Extract and record token usage from response metadata.
- **State fields:** When adding new routing categories, update `AgentState`, add a specialist node, and wire it in `route_to_specialist()` and `build_support_graph()`.
- **Environment config:** AWS uses SAM (`template.yaml` + `samconfig.toml`). GCP uses `.env` files loaded via `python-dotenv` and `deploy.sh` for Cloud Run.

---
> Source: [luigiw/foundry-observability-for-external-agents](https://github.com/luigiw/foundry-observability-for-external-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-29 -->
