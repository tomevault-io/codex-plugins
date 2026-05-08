# AGENTS.md

Guidance for AI coding agents (Claude Code, Cursor, Copilot, etc.) when working with code in this repository.

> **Note:** `CLAUDE.md` is a symlink to this file.

## Repository Overview

A collection of Agent Skills for RisingWave. Skills are packaged instructions that extend AI coding agents with RisingWave-specific expertise: streaming SQL patterns, CDC ingestion, materialized view design, sink configuration, and performance best practices.

## Repository Structure

```
agent-skills/
├── skills/
│   ├── risingwave/
│   │   ├── SKILL.md
│   │   └── references/
│   │       ├── connectors.md
│   │       └── catalog.md
│   └── risingwave-best-practices/
│       ├── SKILL.md
│       └── rules/
│           ├── _sections.md
│           ├── _template.md
│           └── *.md
├── README.md
├── AGENTS.md
├── CONTRIBUTING.md
└── LICENSE
```

## Creating or Editing Skills

Skills follow the [Agent Skills Open Standard](https://agentskills.io/).

### SKILL.md Frontmatter

```yaml
---
name: skill-name              # Lowercase, hyphens only. Must match directory name.
description: "..."            # 1-1024 chars. Primary trigger mechanism — include specific keywords.
license: Apache-2.0
metadata:
  author: RisingWave Labs
  version: "0.1.0"            # Bump on every change to SKILL.md or any file in the skill.
---
```

**Version bumps:** Any change to `SKILL.md` or files in `references/` or `rules/` must bump the `version` in the skill's frontmatter metadata.

### Description Field (Critical)

The description is the **primary trigger mechanism** — Claude uses it to decide when to activate the skill. Include specific RisingWave terms, error messages, and scenarios.

```yaml
# Good — specific triggers
description: Use when working with RisingWave — sources, materialized views, sinks, CDC,
  Kafka ingestion, TUMBLE/HOP/SESSION windows, watermarks, EMIT ON WINDOW CLOSE, port 4566

# Bad — too vague
description: Helps with streaming databases.
```

### Rule File Format (`rules/*.md`)

```markdown
---
title: Action-Oriented Title
impact: CRITICAL|HIGH|MEDIUM-HIGH|MEDIUM|LOW-MEDIUM|LOW
impactDescription: "Quantified benefit"
tags: comma, separated, keywords
---

## Title

1-2 sentence explanation of the problem and why it matters.

**Incorrect:**
```sql
-- Bad: comment explaining the problem
[anti-pattern]
```

**Correct:**
```sql
-- Good: comment explaining the improvement
[correct pattern]
```

Reference: [Doc Page](https://docs.risingwave.com/...)
```

## Contributing New Rules

1. Copy `skills/risingwave-best-practices/rules/_template.md` to `rules/{prefix}-{name}.md`
2. Fill in frontmatter (title, impact, impactDescription, tags)
3. Write explanation with Incorrect/Correct SQL examples
4. Add a reference link to the official RisingWave docs
5. Bump version in `skills/risingwave-best-practices/SKILL.md`
6. Add the rule to the Quick Reference section in `SKILL.md`

## Impact Levels

| Level | Meaning |
|-------|---------|
| CRITICAL | Data correctness issue, or 10× or more improvement |
| HIGH | 2–10× improvement, or significantly impacts scalability |
| MEDIUM-HIGH | 25–100% improvement, or important for specific workloads |
| MEDIUM | 10–25% improvement, or improves maintainability |
| LOW-MEDIUM | 5–10% improvement, or edge-case optimization |
| LOW | Minor improvements |

## Key RisingWave Concepts to Know

- **Port 4566** — PostgreSQL wire protocol endpoint (not 5432)
- **SOURCE** — Streaming connector, no data persistence. For append-only streams (Kafka, S3).
- **TABLE** — Persistent storage. Required for CDC connectors (postgres-cdc, mysql-cdc).
- **MATERIALIZED VIEW** — Continuously maintained query result. The core compute primitive.
- **SINK** — Output connector. Reads from a table or MV, writes to Kafka, Iceberg, DB, etc.
- **Watermark** — Event-time progress marker. Required for `EMIT ON WINDOW CLOSE`.
- **EMIT ON WINDOW CLOSE** — Emits one final result per window when watermark passes window_end.
- **BACKGROUND_DDL** — Non-blocking MV creation for large tables.

## What NOT to Include in Skills

Skills contain only what an AI agent needs. Do NOT add:
- CHANGELOG.md
- INSTALLATION_GUIDE.md
- QUICK_REFERENCE.md
- Marketing copy or feature announcements

---
> Source: [risingwavelabs/agent-skills](https://github.com/risingwavelabs/agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-24 -->
