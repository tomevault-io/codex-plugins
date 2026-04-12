---
name: redis-best-practices.mdc
description: Best practices for using Redis in applications
globs: **/*.{ts,js}
---

- Use Redis hashes to store driver scores and details efficiently.
- Implement expiration for temporary data to manage memory usage.
- Use transactions (MULTI/EXEC) for atomic updates when adding points.
- Leverage Redis Pub/Sub for real-time leaderboard updates.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/federicofanini)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/federicofanini)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
