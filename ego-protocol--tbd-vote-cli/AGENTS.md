
# TBD — Solana Prediction Market for Human Opinions

[TBD](https://www.tbd.vote) is a Solana-based prediction market where users wager on human opinions — what people think, prefer, and predict — rather than purely objective event outcomes.

**The complete agent guide, CLI reference, autonomous loop instructions, and raw HTTP fallback live at https://www.tbd.vote/agents/AGENTS.md — read that file as the source of truth for anything not covered below.**

## Setup

```bash
npm install -g @tbd-vote/cli
tbd-vote login
```

The user must provide their own API key. Never guess or generate credentials.

## Quick command reference

| Command | Purpose |
|---|---|
| `tbd-vote login` | Authenticate with an API key |
| `tbd-vote campaigns list --json` | Browse open opinion-poll campaigns (always use `--json` for parsing) |
| `tbd-vote bet <campaign-id> <option-id>` | Place a bet on a specific option |

## Guidelines

- **Always parse `--json` output** rather than scraping the human-readable format.
- **Always confirm bets with the user** before executing them. Bets are real on-chain transactions and irreversible.
- **Treat opinion markets differently from event markets**: outcomes resolve based on what people believe or vote, not on objective real-world events.
- **For autonomous loops, deeper integration, or anything not covered here, read https://www.tbd.vote/agents/AGENTS.md** — it documents the full autonomous-loop pattern, error handling, and HTTP fallback.

## Links

- Website: https://www.tbd.vote
- AGENTS.md (source of truth): https://www.tbd.vote/agents/AGENTS.md
- CLI repo: https://github.com/ego-protocol/tbd-vote-cli
- npm: https://www.npmjs.com/package/@tbd-vote/cli

---
> Source: [ego-protocol/tbd-vote-cli](https://github.com/ego-protocol/tbd-vote-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-16 -->
