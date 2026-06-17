# Drift — Copilot Development Instructions

## Git workflow — MANDATORY

**All new development must happen on a feature branch. Never commit directly to `main`.**

### Branch naming
```
feature/<short-description>     # new capabilities
fix/<short-description>         # bug fixes
chore/<short-description>       # config, deps, tooling changes
```

### Workflow for every change
1. `git checkout -b feature/<name>` — create branch before writing any code
2. Build the feature in small, logical commits
3. Push the branch: `git push -u origin feature/<name>`
4. Open a Pull Request — do NOT merge it yourself
5. Wait for owner review and approval before merging

### Copilot must ask before:
- Merging or closing a PR
- Deleting files or branches
- Pushing to `main` directly
- Any destructive git operation (`reset --hard`, `push --force`)

---

## Design principles

1. **Modularity over monoliths.** Each concern lives in its own module/class. No file should grow beyond ~200 lines before splitting.
2. **Object-oriented where repeatable.** Abstract base classes define contracts; concrete implementations live in separate files.
   - `data/providers/` — one file per data provider, all implement `MarketDataProvider`
   - `features/` — one file per indicator group, all implement `FeatureComputer`
   - `gates/` — one file per gate (to be built), all implement a common `Gate` interface
3. **Ask before assuming.** When requirements are ambiguous, ask a clarifying question rather than guessing.
4. **Determinism first.** All indicator math and gate logic must be deterministic and auditable. The LLM is a classifier, not a decision-maker.
5. **No signal is better than a bad signal.** Default outputs should be NO_TRADE when uncertain.

---

## Module structure

```
src/drift/
  data/providers/      # MarketDataProvider implementations (yfinance, etc.)
  features/            # FeatureComputer implementations + FeatureEngine coordinator
  gates/               # Deterministic gate layer (session, regime, R:R, cooldown, etc.)
  scoring/             # Regime scoring helpers (feeds into MarketSnapshot)
  ai/                  # LLM client, prompt builder, response parser
  planning/            # TradePlan construction, stop/TP calculation
  output/              # Console & notification rendering — no business logic here
  storage/             # Persistence only — JSONL logger, SQLite (future)
  config/              # Pydantic settings models
```

---

## Planned build order

1. ~~Data layer (yfinance provider)~~ — done
2. ~~Feature engineering (EMAs, VWAP, RSI, ATR, MACD, volume)~~ — done
3. Economic calendar gate — **next**
4. Deterministic gate layer (session, regime, R:R, cooldown, kill-switch)
5. LLM integration (OpenAI structured JSON response)
6. Trade plan constructor + operator output
7. News sentiment enrichment (optional, V2)

---

## Tech stack

- Python 3.11+
- Pydantic v2 — all data models and config
- Rich — terminal output only
- Typer — CLI entrypoints
- yfinance — market data provider (delayed, proxy via NQ=F for MNQ)
- pandas — all indicator math (pure pandas, no C-extension indicator libraries)
- OpenAI SDK — LLM calls (future)

## Testing

- Tests live in `tests/`
- Run with: `pytest`
- Every new module must have at least a smoke test
- Gate logic must have explicit pass/fail case tests

---
> Source: [foo-yay/Drift](https://github.com/foo-yay/Drift) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-17 -->
