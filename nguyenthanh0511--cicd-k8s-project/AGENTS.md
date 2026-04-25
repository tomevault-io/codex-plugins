# PromptAI — Workspace Instructions

Personal AI-assisted learning workspace for a software engineer. Primary language in notes and responses: **Vietnamese**.

## Knowledge Base

All learning notes live in `knowledge-base/`. See [knowledge-base/README.md](../knowledge-base/README.md) for full conventions.

**File naming**: `YYYY-MM-DD_[topic-slug].md`
- Example: `2026-04-08_kubernetes-rbac.md`

**Subfolders**:
| Folder | Purpose |
|--------|---------|
| `concepts/` | Concepts, theory, mental models |
| `bugs/` | Bugs encountered + lessons learned |
| `reviews/` | Code review takeaways |

**After creating any note**, update `knowledge-base/INDEX.md` by appending a row:
```
| YYYY-MM-DD | Short topic title | tag1, tag2 | [filename](path/to/file.md) |
```

## Diagrams

Diagrams use **D2** (`.d2` files in `diagrams/`). Default config:
```d2
vars: {
  d2-config: {
    layout-engine: elk
    theme-id: 300
  }
}
```
See [diagrams/demo-haproxy-flow.d2](../diagrams/demo-haproxy-flow.d2) as a reference pattern.

## Slash Command Workflows

| Command | Action |
|---------|--------|
| `/teach-and-save` | Teach a concept in Vietnamese, then create a file in `knowledge-base/concepts/` and update `INDEX.md` |
| `/bugfix` | Analyze a bug, then optionally save to `knowledge-base/bugs/` |
| `/codereview` | Review code, save lessons to `knowledge-base/reviews/` |

---
> Source: [Nguyenthanh0511/CiCd-K8s-Project](https://github.com/Nguyenthanh0511/CiCd-K8s-Project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-25 -->
