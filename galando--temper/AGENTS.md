
# Source-Driven Development

Write framework-specific code against current official documentation, not stale training data.

For the full source-driven process, apply the `.claude/skills/source-driven-development/SKILL.md` definitions:
- Detect version -> Fetch current docs -> Cite sources -> Surface conflicts
- Authority hierarchy: official docs > type definitions > source code > community > training data
- Use Context7 MCP when available; fall back to web search (label as HEURISTIC)

Apply before writing framework-specific code. Skip for plain logic or known patterns.

---
> Source: [galando/temper](https://github.com/galando/temper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-16 -->
