# RAG Golden Template — Copilot Instructions

## Project Overview
Generic RAG (Retrieval-Augmented Generation) assistant template.
Stack: Python 3.13 · Flask · ChromaDB 1.0.x (Rust backend) · `google-genai` SDK (NOT `google-generativeai`)

**Deploy targets:** Google Cloud Run (primary) · Render (alternative)

## Architecture
```
User → /v1/answer → dual_channel_search() → [law_hits, routine_hits]
                  → _normalize_law_chunk()
                  → call_vertex_ai()        (Jinja2 renders master_prompt.xml)
                  → Gemini Flash            → answer JSON
```

**Key files:**
- `app/server.py` — Flask app, RAG orchestrator, all search logic
- `app/retrieve.py` — `retrieve_dual()` with `doc_type` where-filter
- `app/prompts/master_prompt.xml` — Jinja2 prompt template
- `policy/config.json` — runtime config (assistant_name, disclaimer, etc.)
- `policy/system.md` — system prompt with `{{ASSISTANT_NAME}}` placeholder
- `scripts/embed_and_index.py` — rebuilds `chroma_db/` from sources/
- `scripts/chunk_sources.py` — chunks raw TXT → JSONL

## Client Configuration (Non-negotiables)
- Client name → `policy/config.json` `assistant_name` + env `ASSISTANT_NAME`
- System prompt → `policy/system.md` (use `{{ASSISTANT_NAME}}` and `{{DISCLAIMER}}`)
- Disclaimer → `policy/config.json` `disclaimer`
- ChromaDB collection → env `CHROMA_COLLECTION` (default: `knowledge_base`)
- **NEVER hardcode client names, domain-specific text, or secrets in `app/` files**
- **NEVER commit `.env` or any API keys**

## Critical Gotchas

**ChromaDB 1.0.x:** Incompatible with 0.x databases.
If queries return empty on a valid index: `rm -rf chroma_db/ && python3 scripts/embed_and_index.py`

**Gemini client pattern:**
```python
client = genai.Client(api_key=GEMINI_API_KEY)   # NOT genai.configure()
result = client.models.embed_content(model="models/gemini-embedding-001", ...)
```

**Lazy init:** Gemini + ChromaDB init deferred to first non-health request (`_do_startup()`).
Do not call `init_gemini()` / `init_chroma()` at module level.

**doc_type metadata:** Index MUST be built with `doc_type` field in metadata.
`where={"doc_type": {"$eq": "law"}}` only works if index has this field.

## Indexing Workflow
```bash
# Always bash (not csh)
export GEMINI_API_KEY="your-key"
python3 scripts/embed_and_index.py
git add -f chroma_db/ && git commit -m "rebuild index N vectors" && git push
```

## Code Standards
- Python: type hints, explicit error handling, `logging` over `print`
- File paths via `Path(__file__).resolve().parents[N]`
- JSONL for chunks, JSON for config
- All scripts runnable standalone AND importable as module

## Shell Note
User shell may be **csh**. Use `bash -c "..."` for scripts with redirects.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Bojanbrf) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
