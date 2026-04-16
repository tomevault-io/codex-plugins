# DICS — DR Intelligence Command System

## Project Structure
- `collector/` — Node.js/TypeScript: scheduling, fetching, deduplication, raw storage
- `extractor/` — Python: NLP entity extraction, scoring, embeddings, RAG ingestion
- `shared/schemas/` — JSON schemas for all data models
- `shared/db/` — PostgreSQL migrations (pgvector enabled)
- `config/sources.yaml` — all monitored sources, rate limits, policies
- `alerts/` — Cloudflare Worker webhook endpoint
- `docker-compose.yml` — Postgres + pgvector + Qdrant + Redis

## Conventions
- Every extracted claim MUST reference a Document ID and source URL.
- Collector respects robots.txt and enforces per-domain rate limits.
- All fetched content is SHA-256 hashed for deduplication.
- Audit log is append-only; no record is ever deleted.
- Confidence tiers: official=1.0, multilateral=0.9, trade=0.7, community=0.4
- Outputs are Markdown tables + JSON; no raw HTML in reports.

## Tech Stack
- Collector: Node 20 + TypeScript 5 + `p-queue` + `got` + `@mozilla/readability`
- Extractor: Python 3.12 + spaCy + sentence-transformers + psycopg2 + pgvector
- DB: PostgreSQL 16 + pgvector extension
- Vector: pgvector (primary) or Qdrant (optional)
- Cache/Queue: Redis 7
- Alerts: Cloudflare Workers (TypeScript)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/FTHTrading) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
