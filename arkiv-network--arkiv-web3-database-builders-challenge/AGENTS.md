# Arkiv Web3 Database Builders Challenge — AI Agent Context

This file provides context for AI coding assistants (Claude Code, Cursor, Copilot, Windsurf, Cline, etc.) working with builders participating in the Arkiv Web3 Database Builders Challenge.

## What is this repo?

The official rules, guides, and resources for the **Arkiv Web3 Database Builders Challenge** — a 2-week challenge to build a web3-native application using [Arkiv](https://arkiv.network) as the data layer. Two winners receive $2,500 USD each (paid in USDC).

## Doc Map

| Question | File |
|----------|------|
| What is the challenge? What's the pitch? | [README.md](README.md) |
| What should I build? Requirements? Getting started? | [docs/builders-guide.md](docs/builders-guide.md) |
| Official rules, eligibility, prizes, legal? | [RULES.md](RULES.md) |
| How is my submission scored? | [docs/scoring-rubric.md](docs/scoring-rubric.md) |
| Submission form fields and spec? | [docs/submission-form.md](docs/submission-form.md) *(will be added during the building phase)* |
| Common questions? | [FAQ.md](FAQ.md) |

## Key Facts

- **Prize:** $2,500 USDC per winner (2 winners). Total pool: $5,000.
- **Verticals:** Job Board, Event Platform, or Knowledge Base. Pick one.
- **Scoring weights:** Arkiv integration depth (40%), Functionality (30%), Design & UX (20%), Code quality & docs (10%).
- **All entities expire.** Expiration dates are core to Arkiv, not optional. Shorter-lived = cheaper on mainnet. Builders should choose expiration dates thoughtfully.
- **Testnet only.** All building happens on Arkiv testnet.
- **AI tools allowed.** Copilot, Claude, ChatGPT, etc. are all encouraged.
- **Demo video:** Optional at submission, required for prize claim (2-3 min, English or Spanish).
- **Open source required.** MIT, Apache 2.0, or equivalent.

## Arkiv Resources

- **Documentation:** https://arkiv.network/docs
- **Developer Portal:** https://arkiv.network/dev
- **TypeScript SDK:** https://arkiv.network/getting-started/typescript
- **Python SDK:** https://arkiv.network/getting-started/python

### Current Testnet: Kaolin (as of Feb 25, 2026)

Mendoza testnet is unstable. **Use Kaolin for all building:**

| | |
|---|---|
| **Network ID** | `60138453025` |
| **HTTP RPC** | `https://kaolin.hoodi.arkiv.network/rpc` |
| **WebSocket RPC** | `wss://kaolin.hoodi.arkiv.network/rpc/ws` |
| **Standard Bridge** | `0x6db217C596Cd203256058dBbFcA37d5A62161b78` |
| **TS/JS SDK version** | `@arkiv-network/sdk` v0.6.0 or newer |

## Minimum Technical Requirements

Every submission must:
- Store all core data as Arkiv entities (not a traditional database)
- Use wallet-based ownership (creators own their data)
- Have at least 2 entity types with a relationship between them
- Use queryable attributes for filtering or search
- Set rational expiration dates on entities
- Allow public read access (no wallet needed to browse)
- Be open source on GitHub with a working demo link and README

## When Helping Builders

- **Entity design:** Point to `docs/builders-guide.md` for suggested entity patterns per vertical. Don't invent data models — guide builders to think about entity types, relationships, queryable attributes, and expiration.
- **Expiration:** All Arkiv entities expire. Don't use the term "TTL" — Arkiv calls it "expiration dates." Help builders choose different expiration durations for different entity types (e.g., job listings: 30-90 days, company profiles: longer).
- **Scoring:** The rubric in `docs/scoring-rubric.md` is published and transparent. If a builder asks how to score well, point them to the specific sub-criteria.
- **Rules questions:** Always reference `RULES.md` for anything about eligibility, prizes, deadlines, or legal terms. Don't paraphrase legal language.
- **Tech stack:** Arkiv is the required data layer. Everything else (frontend, styling, wallet lib, hosting) is the builder's choice. Suggest what fits their experience.
- **Support:** Direct builders to #builders-challenge on Discord for Arkiv-specific issues.

---
> Source: [Arkiv-Network/arkiv-web3-database-builders-challenge](https://github.com/Arkiv-Network/arkiv-web3-database-builders-challenge) — distributed by [TomeVault](https://tomevault.io/claim/Arkiv-Network).
<!-- tomevault:4.0:agents_md:2026-04-17 -->
