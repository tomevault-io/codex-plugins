# SAP CodeJam: Code-Based AI Agents

Hands-on workshop for building multi-agent AI systems on SAP BTP using **CrewAI** and **LiteLLM** connected to **SAP Generative AI Hub** (AI Core). The scenario: investigate an art heist by orchestrating specialized agents that analyse evidence, appraise stolen items, and identify the culprit.

## Architecture

Three-agent sequential crew in `project/Python/`:

| Agent | Tool | Purpose |
|---|---|---|
| Appraiser | `call_rpt1()` | Predict item categories & insurance values via SAP-RPT-1 ML model |
| Evidence Analyst | `call_grounding_service()` | RAG queries over evidence documents via SAP Grounding Service |
| Lead Detective | _(none)_ | Synthesise findings and name the culprit |

Key files in `project/Python/solution/`:
- `investigator_crew.py` — `@CrewBase` class; agents and tasks defined with `@agent`, `@task`, `@crew` decorators; tools use `@tool("name")`
- `main.py` — entry point; calls `crew().kickoff(inputs={...})`
- `rpt_client.py` — OAuth2 client for SAP-RPT-1 (client-credentials grant, Bearer auth)
- `payload.py` — structured art-item data with `[PREDICT]` placeholders for RPT-1
- `config/agents.yaml`, `config/tasks.yaml` — YAML definitions (method names must match decorator names)

Evidence documents (plain text, loaded into the Grounding Service pipeline) are in `exercises/data/documents/`.

## Build & Run

```bash
# Activate virtual env (Windows)
.\env\Scripts\Activate.ps1

# Install dependencies (only needed once)
pip install litellm crewai python-dotenv

# Run the crew
python main.py
```

Requires a `.env` file in `project/Python/starter-project/` (copy structure from the exercise docs):

```
AICORE_CLIENT_ID=
AICORE_CLIENT_SECRET=
AICORE_AUTH_URL=
RPT1_DEPLOYMENT_URL=
AICORE_RESOURCE_GROUP=
AICORE_BASE_URL=
```

## Conventions

- **CrewAI YAML config**: agent/task names in `agents.yaml` / `tasks.yaml` must exactly match the Python method names decorated with `@agent` / `@task`.
- **Tool pattern**: tools are plain functions decorated with `@tool("Descriptive Name")`. Return error messages as strings so the LLM can handle failures gracefully.
- **LLM model strings**: use `sap/<model-name>` format (e.g. `sap/gpt-4o`) matching deployments in SAP AI Launchpad.
- **Process**: always `Process.sequential` — tasks pass outputs as context to the next task in order.
- **RPT-1 payload**: `[PREDICT]` string is the placeholder for values to be inferred; schema (dtype, categories, value ranges) must be exact.

## Pitfalls

- **Hardcoded Grounding pipeline ID** in `call_grounding_service()` — replace with your own vector DB pipeline ID from SAP AI Launchpad before running.
- **Token refresh**: `RPT1Client` fetches the OAuth token once at init; long-running crews may hit expiry — re-instantiate if needed.
- **No `.env` validation** at startup — credential errors only surface on the first API call.
- **YAML / decorator name mismatch** causes silent CrewAI failures with no clear error message.
- **Grounding pipeline must be pre-loaded** with the evidence documents; empty pipelines return no results and agents will hallucinate.

---
> Source: [SAP-samples/codejam-code-based-agents](https://github.com/SAP-samples/codejam-code-based-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-02 -->
