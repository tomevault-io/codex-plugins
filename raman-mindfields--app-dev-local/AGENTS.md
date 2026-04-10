Purpose
=======
Make AI coding assistants productive quickly in this repository by summarizing architecture, run/test/deploy workflows, configuration patterns, and notable code conventions.

Quick facts
-----------
- Language: Python backend (Quart), TypeScript/React frontend (Vite). Frontend build outputs to `static/` and is served by the backend Flask/Quart app.
- Entry point (backend): `app.py` (registers blueprint `routes` and exposes REST endpoints used by the frontend).
- Runtime: Python 3.11+ is expected (see Dockerfile / README). The app uses Async Azure OpenAI client and optional Azure services (Cognitive Search, CosmosDB, Promptflow, etc.).
- Frontend: `frontend/` contains Vite + React app. Built artifacts are copied to `static/` and shipped with the Python app.

High-level architecture (what to know)
-------------------------------------
- app.py: central HTTP server using Quart. It initializes Azure OpenAI client (async), optional CosmosDB history client, and exposes these endpoints:
  - `/conversation` - single-shot streaming or non-streaming chat
  - `/history/*` - conversation history CRUD backed by CosmosDB
  - `/frontend_settings` - returns UI settings read from `backend/settings.py`
- backend/settings.py: Pydantic-based settings loader. Environment variables drive behavior. Key concepts:
  - `app_settings.azure_openai` controls model, endpoint, embedding settings and function-call integration.
  - `app_settings.base_settings.datasource_type` selects a datasource implementation (Azure Cognitive Search, CosmosDB vector index, Elasticsearch, Pinecone, MongoDB, Azure SQL).
  - Chat history is optional and controlled by `AZURE_COSMOSDB_*` env vars.
- Frontend <-> Backend flow:
  - Frontend calls `/conversation` and `/history/*` endpoints. The backend constructs model payloads (see `prepare_model_args` in `app.py`) and may add `extra_body.data_sources` when a datasource is configured.
  - When `AZURE_OPENAI_STREAM` is true, responses are streamed as NDJSON from the backend.
- Function calling: When `AZURE_OPENAI_FUNCTION_CALL_AZURE_FUNCTIONS_ENABLED` is true, backend fetches remote function definitions from a configured Azure Functions URL and will execute them during streaming or non-streamed completions.

Developer workflows (how to run / build / test)
---------------------------------------------
- Local development (recommended):
  1. Copy `.env.sample` -> `.env` and fill required variables (see README.md for full list). This repository reads `.env` via pydantic settings in `backend/settings.py`.
  2. Build & run helper: `start.cmd` (Windows) or `start.sh` (Bash) — these build the frontend, install backend dependencies, then start the Quart app. Use the PowerShell-friendly `start.cmd` on Windows.
  3. Open http://127.0.0.1:50505 to access the app.
- Quick run (backend only): Use the VS Code debug configuration or run the backend directly with Python (ensure frontend static files exist):
  - python -m app  (the repository expects `app` imported as Quart app; prefer using `start.cmd` which runs build steps)
- Frontend dev: from `frontend/` you can run standard Vite dev commands (package.json present). Built frontend artifacts are placed under `static/` for production.
- Tests: tests live in `tests/`. Run unit and integration tests using the project's test runner (see `requirements-dev.txt`); typical command: `pytest` after installing dev requirements.

Configuration and secrets
-------------------------
- `.env` is the single source of truth for config. `backend/settings.py` maps env vars to typed settings. Look at `DOTENV_PATH` logic when debugging env loading.
- Sensitive fields are excluded from serialized payloads in `backend/settings.py` (e.g., embedding keys, API keys). When constructing model payloads the backend masks secrets before logging.

Project-specific conventions and patterns
---------------------------------------
- Settings: Prefer environment-driven config via pydantic models in `backend/settings.py`. When adding new runtime flags, add them to the appropriate Settings class and use `model_validator`/`field_validator` for transformation.
- Datasources: `DatasourcePayloadConstructor` subclasses implement `construct_payload_configuration(request=...)` to produce the datasource payload that is included in model requests. To add a new datasource, implement the constructor pattern used by `_AzureSearchSettings` or `_ElasticsearchSettings`.
- Streaming: The app supports both streaming and non-streaming responses. Streaming uses NDJSON generator helpers in `backend/utils.py` referenced from `app.py`. When changing response formatting, update both streaming and non-streaming helpers.
- Function calls: Remote function metadata is fetched at startup and appended to the model payload under `tools`. The code expects function results returned synchronously from remote Azure Functions endpoints.

Integration points and external dependencies
-------------------------------------------
- Azure OpenAI (core dependency). Configuration keys: `AZURE_OPENAI_*` (see README + `backend/settings.py`). The code uses AsyncAzureOpenAI client.
- Optional or used services:
  - Azure Cognitive Search (AZURE_SEARCH_*)
  - Cosmos DB (AZURE_COSMOSDB_*) for chat history
  - Promptflow (PROMPTFLOW_*) as an alternative model pipeline
  - Elasticsearch, Pinecone, AzureMLIndex, MongoDB, Azure SQL (supported via `datasource_type`)
  - Microsoft Defender integration (MS_DEFENDER_ENABLED)

Code locations to inspect for behavior
-------------------------------------
- Request / response construction: `app.py` (prepare_model_args, send_chat_request, stream_chat_request, conversation_internal)
- Settings and env parsing: `backend/settings.py`
- Conversation storage: `backend/history/cosmosdbservice.py`
- Client utilities and formatters: `backend/utils.py` (NDJSON, streaming helpers, mask secrets)
- Frontend UI and API calls: `frontend/src/` (chat pages, components like `Answer/Answer.tsx` show how citations and feedback are rendered)

Examples of repo-specific patterns (copy/paste friendly)
-----------------------------------------------------
- How the backend masks secrets before logging:
  - app.py -> prepare_model_args: model_args_clean hides `key`, `connection_string`, `embedding_key`, `encoded_api_key`, `api_key`.
- How a datasource payload is constructed when calling the model:
  - In `prepare_model_args`, when `app_settings.datasource` exists, it calls `app_settings.datasource.construct_payload_configuration(request=request)` and places the result into `model_args['extra_body']['data_sources']`.

What to avoid/change carefully
------------------------------
- Don't change settings parsing silently. The pydantic models include validators that accept both strings and JSON-encoded env vars (see `deserialize_tools`, `split_contexts`).
- Keep streaming and non-streaming formatting consistent: format_stream_response vs format_non_streaming_response in `backend/utils.py`.

If you change behavior, run these quick checks
--------------------------------------------
1. Update `.env` (copy from `.env.sample`) and set minimal required keys (AZURE_OPENAI_MODEL, AZURE_OPENAI_ENDPOINT/RESOURCE, AZURE_OPENAI_KEY or Entra ID environment).  
2. Run `start.cmd` (Windows) or `start.sh` (Linux/Mac) and verify the app serves at http://127.0.0.1:50505
3. Verify frontend calls `/conversation` successfully (use browser devtools network tab).  
4. If changing history logic, run `/history/ensure` to validate CosmosDB connectivity.

Where to update this document
-----------------------------
- Keep this file in `.github/copilot-instructions.md`. When adding new components (e.g., a new datasource), add quick notes and reference example classes in `backend/settings.py` and any new utility functions in `backend/utils.py`.

Questions for the maintainers (if unclear)
----------------------------------------
- Which Python runtime should CI assert (3.11 vs 3.12)? README implies 3.11 but CI/testing matrix isn't present.
- Are there any additional local dev scripts to set up managed identity emulation or mocking for Azure services?

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Raman-Mindfields)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Raman-Mindfields)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
