## Purpose

This file gives concise, repository-specific guidance for AI coding agents working on this project so they can be productive immediately.

**Big Picture**:
- **Data flow**: scrapers/parsers produce JSON files named `concert_events_*.json` at the repo root. Notebooks read the latest JSON (see `ingest.ipynb` -> `load_latest_events`) and push to a local Meilisearch index named `events`.
- **Components**: interactive notebooks (`ingest.ipynb`, `parsing.ipynb`) are the primary developer entry points; there is no separate service code or tests in the repo.
- **Search backend**: Meilisearch stores indexes under `data/data.ms/` (do not delete). The Python client used is the `meilisearch` package (installed via `environment.yml`).

**Quick setup / workflows**:
- Create the Conda env listed in `environment.yml`:

```bash
conda env create -f environment.yml
conda activate eventing
```
- Start Meilisearch (recommended: Docker, mounts `data/data.ms` to persist DB):

```bash
docker run --rm -it -p 7700:7700 \
  -v "$PWD/data/data.ms:/data.ms" \
  -e MEILI_MASTER_KEY="<your-key>" \
  getmeili/meilisearch:latest \
  meilisearch --db-path /data.ms --master-key "$MEILI_MASTER_KEY"
```
- Open `ingest.ipynb` in JupyterLab and run the cells to index the latest `concert_events_*.json` into index `events`.

**Important repo-specific patterns & conventions**:
- File naming: search for `concert_events_*.json` and assume `load_latest_events(basepath="concert_events")` will pick the most recent file.
- Indexing: code in `ingest.ipynb` calls `index.add_documents(events, primary_key="id")`. The code also calls `index.update_searchable_attributes(...)`, `update_filterable_attributes(...)`, and `update_sortable_attributes(...)` — follow this pattern when adding new fields.
- Index name and keys: use index name `events` and document primary key `id` (existing code assumes both).
- Meilisearch configuration: the notebook uses a hard-coded URL and API key. Prefer using environment variables `MEILI_URL` and `MEILI_API_KEY` when modifying code; example snippet:

```python
import os
client = meilisearch.Client(os.getenv('MEILI_URL', 'http://localhost:7700'), os.getenv('MEILI_API_KEY'))
```

**Search & query examples (from `ingest.ipynb`)**:
- Faceted search sample:

```python
facet_fields = ["location","status_kind","price_eur","date","band"]
result = index.search(
  "sonic",
  {
    "filter": 'price_eur < 50 AND status_kind != "ausverkauft"',
    "facets": facet_fields,
    "sort": ["date:desc"],
    "limit": 1000,
  }
)
```

**Files & locations to reference**:
- `ingest.ipynb` — primary indexing and search examples; contains `load_latest_events()` and indexing flow.
- `parsing.ipynb` — parsing / scraping flows that generate `concert_events_*.json`.
- `environment.yml` — environment and Python dependency list (includes `meilisearch` pip package).
- `data.ms/` — Meilisearch DB directory; treat as persistent data (do not remove) and mount into Docker when running Meilisearch.

**Integration points & external dependencies**:
- Meilisearch: local instance expected on `http://localhost:7700` by default; the notebooks assume a master-key is set.
- Python packages: `meilisearch` (client), `pandas`, `numpy`, `beautifulsoup4`, `requests`, `spotipy`, `musicbrainzngs` (pip). See `environment.yml`.

**Conventions agents should follow when editing**:
- Prefer non-destructive edits to notebooks: when adding code examples, add clearly labeled cells or create small helper scripts (e.g., `scripts/reindex.py`) instead of editing parsing logic in-place.
- Do not hard-code secrets into files. If you find the API key present in notebooks, replace with `os.getenv(...)` and document the change in the same commit.
- When modifying indexing fields, update all three attribute lists in `ingest.ipynb`: searchable, filterable, sortable (as appropriate).

**What NOT to change** (unless the user asks):
- Do not delete or reinitialize `data.ms/` — it holds the Meilisearch DB.
- Do not rename the `concert_events_*` files; consumers expect that naming pattern.

If anything in this file is unclear or you'd like examples turned into small standalone scripts, tell me which parts to expand and I'll update the instructions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andreaseri)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/andreaseri)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
