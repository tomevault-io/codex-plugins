# ISTQB RAG Companion - Project Guidelines

## Scope
These instructions apply to all work in this repository.

## Architecture
- Build and maintain a pure Python RAG pipeline. Do not introduce RAG frameworks such as LangChain or LlamaIndex.
- Keep the provider pattern for LLM access through `get_llm()` with two backends: Ollama and enterprise OpenAI-compatible API.
- Keep retrieval deterministic with ChromaDB metadata filters (`capitulo` and `k_level`) on every query.
- Keep deterministic validation before returning any generated question.

## Required Data Contracts
- Question output must be strict JSON with keys: `stem`, `options` (4 entries), `correct` (A/B/C/D), `explanation`.
- Chroma metadata per chunk must include: `capitulo`, `seccao`, `k_level`, `tipo_conteudo`.

## Build And Run
- Prefer `uv` for dependency and environment management.
- Install runtime dependencies with `uv`:
  - `uv init`
  - `uv add chromadb streamlit requests pymupdf python-dotenv`
- Alternative without `uv`:
  - `pip install chromadb streamlit requests pymupdf python-dotenv`
- Pull Ollama models when using local backend:
  - `ollama pull llama3.1:8b`
  - `ollama pull bge-m3`
- Ingest syllabus data:
  - `uv run python ingest.py --pdf data/syllabus.pdf`
- Run the app:
  - `uv run streamlit run app.py`

## Test
- Run tests with:
  - `uv run pytest tests/ -v`

## Coding Conventions
- Use type hints on function signatures.
- Use docstrings for public functions.
- Use f-strings for formatting.
- Keep constants in `UPPER_SNAKE_CASE` in `config.py`.
- Use `logging` instead of `print()`.
- Keep ChromaDB query logic centralized in `rag_pipeline.py`.

## Reliability Rules
- Wrap LLM calls and JSON parsing in explicit error handling with useful messages.
- If validation fails, retry generation up to 2 times before surfacing an error.
- Do not bypass validation before UI rendering.

## Common Pitfalls
- Missing models (`llama3.1:8b`, `bge-m3`) will break local generation and embeddings.
- Missing `data/syllabus.pdf` will break ingestion.
- Misconfigured `.env` values (`LLM_BACKEND`, API settings) can silently route to wrong backend.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sudo-jpsilva)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/sudo-jpsilva)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
