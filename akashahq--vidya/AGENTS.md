# Vidya — Copilot Instructions

## Project Overview

Vidya is a hybrid memory plugin for OpenClaw agents. It combines a LanceDB vector store with Voyage AI embeddings and reranking to provide semantic + keyword hybrid search over agent memory.

## Architecture

```
index.ts              → Plugin entry point (OpenClaw plugin interface)
src/config.ts         → Configuration schema and defaults
src/embeddings/       → Embedding providers (Voyage AI, OpenAI, Jina, etc.)
src/retrieval.ts      → Hybrid retrieval logic (semantic + keyword search, reranking)
src/utils.ts          → Shared utilities (URL normalization, dedup helpers)
test/unit.test.ts     → Unit tests
```

## Build & Test

```bash
# Install dependencies
npm install

# Type check
npx tsc --noEmit

# Run unit tests
npx tsx --test test/unit.test.ts

# Run tests with coverage
npx c8 --reporter=lcov --reporter=text npx tsx --test test/unit.test.ts
```

> **Warning:** After modifying `.ts` files, clear the jiti cache to avoid stale builds:
> ```bash
> rm -rf /tmp/jiti/
> ```

## Code Style

- **TypeScript strict mode** — all code must pass `tsc --noEmit` with strict enabled.
- **Conventional Commits** — commit messages follow the Conventional Commits specification (e.g., `feat:`, `fix:`, `chore:`).
- **Shared utilities** — reusable helpers live in `src/utils.ts` (URL normalization via `normalizeBaseUrl()`, deduplication helpers). Use them instead of reimplementing.
- **No hardcoded credentials** — API keys and secrets must come from config or environment variables, never committed to source.

## PR Review Focus

When reviewing pull requests, pay special attention to:

1. **Security** — No credential exposure in code, logs, or test fixtures. All secrets via env vars or config.
2. **Correctness** — Search relevance logic, score calculation, and ranking must be accurate. Verify hybrid search weighting.
3. **Error handling** — Graceful fallback when LLM, embedding, or reranking services are unavailable. The plugin should degrade, not crash.

---
> Source: [AkashaHQ/Vidya](https://github.com/AkashaHQ/Vidya) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-04 -->
