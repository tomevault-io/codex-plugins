
# Self-learning

Make yourself self-improving: when you earn a reusable **golden path**, capture
it so the next session starts knowing it instead of rediscovering it.

## Recognize the moment

- A task only worked after several attempts, wrong turns, or a correction.
- You found project facts you didn't know up front: where creds/env vars live,
  a non-obvious command, a required sequence, a gotcha that defies the obvious.
- An operational workflow likely to recur — reach the dev/prod DB, deploy, run
  migrations, seed data, verify a change live.
- The user says "remember this" / "don't make me re-explain this next time".

Act on the cue immediately — **don't ask permission first**. Then tell the user
what you saved and where.

## Capture it as a rule

Write a new rule at `.cursor/rules/learned/<short-name>.mdc`:

```mdc
---
description: <what it does AND when to use it — this is how Cursor reloads it later>
# or, if path-specific instead of description-triggered:
# globs: ["path/glob/**"]
---

# <title>

**Goal:** <the recurring task this serves>

## Steps
1. <exact command / path; keep the required order if it matters>

## Gotchas
- <non-obvious fact; where a secret lives — never the value>

## What didn't work
- <approach you ruled out, and why>
```

Capture the **procedure** (commands, paths, order, gotchas), not a one-off
answer, plus the **failures** so next time skips the dead-ends.

## Rules

- **Never write secret values** (tokens, passwords, connection strings, keys).
  Record only *where* they live (env var name, config file, secret manager).
- A **one-line fact** → put it in project notes/memory, not a whole rule.
- A genuine **one-off** → skip it.

---
> Source: [Kulaxyz/self-learning-skills](https://github.com/Kulaxyz/self-learning-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-30 -->
