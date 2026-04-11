# Ollama Chat Streamer – Copilot Instructions

## Overview & Architecture

The **Ollama Chat Streamer** is a CLI tool that provides an interactive chat experience with Ollama language models. The main components are:

- **`stream_chat.py`** – the entry‑point that parses CLI arguments, loads optional context, drives the chat loop, and optionally persists conversations to PostgreSQL.
- **`db.py`** – thin wrapper around **`asyncpg`** / **`psycopg2`** that creates the `conversations` table, saves, updates and retrieves conversation records.
- **`setup_db.py`** – a helper script used during development to create the tables and optionally reset the database.
- **Docker** – `docker-compose.yml` runs two services:
  - **`chat`** – builds the Python image and shares the host network so it can reach a locally‑running Ollama server (`http://localhost:11434`).
  - **`postgres`** – a PostgreSQL instance used for optional persistence.

The data flow is simple:

1. **User input** → `stream_chat.py` appends a _user_ message to the `messages` list.
2. **Model call** – `_run_chat_turn` streams a response from Ollama. If `--experimental-websearch` is enabled, the model may request the `web_search` tool, which is executed by `chat_with_tools`.
3. **Response handling** – the assistant’s reply is printed, logged to the destination file (`--dest`) and, when `--persist-to-db` is set, saved/updated in PostgreSQL via `db.py`.
4. **Context loading** – `--context` can pull files (via `load_context_files`) or entire conversation history from the DB (`load_context_from_database`). The loaded text is injected as a _system_ message before the chat loop starts.

## Critical Developer Workflows

### Local Development (no Docker)

```bash
# Install dependencies
pip install -r requirements.txt

# Run the chat directly (Ollama must be running on the host)
python stream_chat.py [options]
```

_The most common flags_:

- `--model` – choose the primary model.
- `--model-fallbacks` – comma‑separated list of fallback models.
- `--experimental-websearch` – let the model decide when to search the web.
- `--context <path>` – load a file, directory, or `db` for conversation history.
- `--persist-to-db` – store the conversation in PostgreSQL (requires `DATABASE_URL`).

### Docker‑Based Development

```bash
# Build the image (only needed after dependency changes)
docker compose build

# Bring up the services (Postgres + chat container sharing host network)
docker compose up -d

# Run a one‑off chat session (the container will exit when you type exit/quit)
docker compose run --rm chat [options]
```

**Important:** The `chat` service uses `network_mode: host` so it can reach the host‑side Ollama server at `http://localhost:11434`. On macOS/Windows you must set `OLLAMA_HOST=http://host.docker.internal:11434` in the `.env` file instead.

### Database Management

- **Initialize / reset** – `python setup_db.py` creates the tables. Use `python setup_db.py reset` to drop and recreate the DB (useful after schema changes).
- **Inspect** – `python setup_db.py info` prints the number of stored conversations and the latest timestamps.
- **Direct queries** – you can connect with any PostgreSQL client using the `DATABASE_URL` value.

## Project‑Specific Conventions & Patterns

1. **Environment‑first configuration** – every CLI flag has a matching `ENV` variable (see the table in `README.md`). The precedence order is **CLI > ENV > default**.
2. **Streaming output** – the chat loop prints tokens as they arrive (`print(part, end="", flush=True)`). This is intentional to give a real‑time feel.
3. **Tool schema** – the only built‑in tools are `web_search` (DuckDuckGo) and `read_json_file`. Adding a new tool requires extending `get_tools()` and handling it in `execute_tool_call()`.
4. **Context injection** – loaded context is always added as a _system_ message with the prefix `You have access to the following context:`. This pattern is relied upon by downstream prompts.
5. **Database schema** – the `conversations` table stores `model`, `flags` (JSONB), `messages` (JSONB), and timestamps. Flags capture the runtime configuration (experimental mode, websearch, fallbacks, etc.) so a conversation can be reproduced later.
6. **Retry logic** – `_retry_call` and `_stream_chat_with_retry` implement exponential back‑off with jitter. All network calls to Ollama go through these helpers.

## Integration Points & External Dependencies

- **Ollama** – the LLM server. It must be reachable at the URL defined by `OLLAMA_HOST`. The default is `http://localhost:11434`.
- **DuckDuckGo Search** – used by the `web_search` tool (`ddgs` package). The tool is only imported when `--experimental-websearch` is set.
- **PostgreSQL** – optional persistence. The Docker compose file provides a `postgres` service; otherwise you can point `DATABASE_URL` at any reachable PostgreSQL instance.
- **Python dependencies** – listed in `requirements.txt` (`ollama`, `python-dotenv`, `psycopg2-binary`, `asyncpg`, `ddgs`). The `psycopg2-binary` package is required for the synchronous DB path used by the CLI.

## Syntax Checking & Testing

- **Static analysis** – run `python -m pyflakes .` or `flake8` to catch unused imports and syntax issues. Be sure to use Pylance or a similar tool in your IDE for real‑time feedback and MyPy for type checking.
- **Unit tests** – currently limited to `test_db.py`, which verifies the DB module functions. Run with `pytest` or your preferred test runner.
- **Manual validation** – after refactors, run `python stream_chat.py --model llama3 --experimental` and have a short chat to ensure behaviour matches the pre‑refactor version.

## Frequently Used Commands (Copy‑Paste)

```bash
# Run with a specific model and fallback list
python stream_chat.py --model llama3 --model-fallbacks "mistral,codellama"

# Enable web search and load a docs directory as context
python stream_chat.py --experimental-websearch --context ./docs --context-grep "md,txt"

# Persist to DB and load previous conversations as context
export DATABASE_URL="postgresql://postgres:postgres@localhost:5432/chatdb"
python stream_chat.py --persist-to-db --context db

# Docker one‑off run with custom flags
docker compose run --rm chat --model gemma --experimental-websearch
```

---

**Tip for AI agents:** When you need to modify or extend functionality, prefer adding a new module under `core/` (e.g., a new tool) and wiring it in `stream_chat.py` via the existing helper functions. Keep the entry‑point thin – it should only orchestrate parsing, context loading, and the chat loop.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/maxxheth)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/maxxheth)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
