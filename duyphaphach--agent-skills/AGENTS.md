
# Agent Skills Authoring Rules

All skill files must conform to the [Agent Skills specification](https://agentskills.io/specification).

## SKILL.md Frontmatter

Every `SKILL.md` must start with YAML frontmatter containing exactly these fields (in this order):

```yaml
---
name: skill-name          # required
description: >-           # required
  What the skill does and when to use it.
license: MIT              # optional
compatibility: ...        # optional — only if environment requirements exist
metadata:                 # optional — arbitrary key/value pairs
  author: ...
  version: "1.0.0"
allowed-tools: Read Write Glob Grep LS   # optional — space-delimited, experimental
---
```

### Field rules

| Field | Required | Constraints |
| ----- | -------- | ----------- |
| `name` | Yes | 1–64 chars. Lowercase letters, numbers, and hyphens only. No leading/trailing hyphens. No consecutive hyphens (`--`). Must match the parent directory name. |
| `description` | Yes | 1–1024 chars. Must describe **what** the skill does **and when** to use it. Include keywords agents use to identify relevant tasks. |
| `license` | No | License name or reference to a bundled license file. |
| `compatibility` | No | 1–500 chars. Only include when the skill has specific environment requirements. |
| `metadata` | No | Map of string → string. Keys should be reasonably unique. |
| `allowed-tools` | No | Space-delimited list of pre-approved tool names (experimental). |

**No other frontmatter fields are permitted.**

## Body content

- Write plain Markdown — no JSX or HTML components.
- Recommended sections: step-by-step instructions, examples, edge cases.
- Keep `SKILL.md` under 500 lines. Move detailed reference material to `references/`.
- Reference other files using relative paths from the skill root: `[guide](references/guide.md)`.
- Avoid deeply nested reference chains.

## Directory structure

```
packages/skills/<skill-name>/
├── SKILL.md              # required
├── references/           # optional — detailed docs loaded on demand
├── scripts/              # optional — executable helpers
└── assets/               # optional — static resources
```

## Naming conventions

- Skill directory and `name` field must match exactly.
- Reference files: focused markdown, one domain per file, with YAML frontmatter (`name`, `description`, `tags`).
- Script files: self-contained, include helpful error messages.

## Validation

Run `skills-ref validate ./packages/skills/<skill-name>` to validate before committing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/duyphaphach) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
