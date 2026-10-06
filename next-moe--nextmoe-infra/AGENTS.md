
# Executor dispatch

**If you were started by `.claude/skills/dispatch-cursor/dispatch.sh` or
`.claude/skills/dispatch-grok/dispatch.sh`, you are the executor: ignore this rule and follow
your task book.**

An orchestrator session drafts task books and hands implementation and investigation to a local
executor CLI; it keeps adjudication, git, databases, production and final acceptance.

- **cursor-agent** is the default executor since 2026-09-17: `.claude/skills/dispatch-cursor/`.
  It runs inside Cursor's kernel sandbox, which turns itself off without a word under `--force`,
  `approvalMode: "unrestricted"` or an allow-listed command. Dispatch only through its
  `dispatch.sh`, and trust a run only when `check.sh` finds the preflight line.
- **grok** is paused (balance exhausted, 402): `.claude/skills/dispatch-grok/`.

Read the chosen skill's `SKILL.md` before the first dispatch of a session.

---
> Source: [next-moe/nextmoe-infra](https://github.com/next-moe/nextmoe-infra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-10-06 -->
