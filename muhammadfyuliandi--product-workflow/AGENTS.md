# GitHub Copilot Instructions
> Auto-applied to every Copilot Chat request in this repo.
> Source of truth for Copilot. Other agents use AGENTS.md directly.

---

## What this repo is
A document-first product development system. Features are fully specced before any code is written.
Copilot's job here is primarily **doc generation**, not code generation — unless a tech spec exists.

---

## Workflow Copilot must follow

```
Idea → Discovery → PRD → Roadmap → Tech Spec → Tasks → (then code)
```

If a user asks Copilot to write code for a feature that has no doc in `docs/`, say:
> "I don't see a spec for this feature yet. Want me to generate one first, or proceed anyway?"

---

## Doc generation behavior

When generating any doc:
1. Read the relevant agent file from `.github/agents/` first
2. Read any existing docs for this feature in `docs/`
3. Generate using the template in the agent file
4. End with `## Handoff Summary` — 3–5 bullet points of what the next agent needs to know
5. Save to the correct folder (product → `docs/product/`, engineering → `docs/engineering/`)

---

## Code generation behavior (when a spec exists)

- Use entity names, endpoint names, variable names exactly as written in the tech spec
- Match the tech stack specified in the tech spec — do not introduce new dependencies
- If the spec is ambiguous, add a `// TODO: clarify with tech spec` comment, don't guess
- Tests go in `tests/` and must cover every endpoint listed in the tech spec

---

## Formatting rules (all docs)

```yaml
---
title: "Feature Name — Doc Type"
date: YYYY-MM-DD
owner: ""
status: draft | review | approved
---
```

- Headings: sentence case, never Title Case
- Lists over paragraphs for requirements
- Every doc ends with `## Open Questions` and `## Handoff Summary`
- Tables for comparisons, metrics, and data models

---

## What Copilot should never do

- Never generate code that bypasses authentication
- Never hardcode API keys or secrets — use environment variables
- Never create files outside `docs/` or `src/` without asking
- Never silently resolve a conflict between docs — flag it in Open Questions

---

## Agent files (Copilot should read before generating each doc type)

| Doc type | Agent file |
|----------|-----------|
| Discovery | `.github/agents/product-discovery.agent.md` |
| PRD | `.github/agents/prd-writer.agent.md` |
| Roadmap | `.github/agents/roadmap-writer.agent.md` |
| Tech Spec | `.github/agents/tech-spec-writer.agent.md` |
| Tasks | `.github/agents/task-breakdown.agent.md` |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/muhammadfyuliandi)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/muhammadfyuliandi)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
