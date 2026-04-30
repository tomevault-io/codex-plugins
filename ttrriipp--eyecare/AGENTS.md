# Always Read the Project Plan

Before responding to any user prompt, you MUST read the plan file:

```
.cursor/plans/backend_foundation_setup_5b4f9e9c.plan.md
```

This plan contains the full architecture, database schema, module breakdown, development timeline, key decisions, and scope for the Optical Management System backend. Use it as the source of truth for all implementation work.

- Check todo statuses to know what has been completed and what is pending.
- Follow the directory structure, naming conventions, and layered architecture defined in the plan.
- Respect the module development pattern (migration → model → enum → service → controller → form requests → API resources → policy → routes → seeder).
- Do not deviate from the database schema or key decisions unless the user explicitly requests changes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ttrriipp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-13 -->
