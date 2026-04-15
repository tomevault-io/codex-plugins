# Multi-Agent Collaboration System

This workspace uses a multi-agent collaboration pattern where specialized agents work together through file-based communication.

## System Overview (Behavioral details live in `AGENTS.md`)

This file provides a high-level overview and file map for the multi-agent system. For authoritative behavior, workflows, and quality standards, see `AGENTS.md`. For exact file formats, see `.github/instructions/*.instructions.md`.

## File Locations

```
.github/
├── copilot-instructions.md        # This file - system overview
├── instructions/                  # Format specifications (auto-applied)
│   ├── inbox-format.instructions.md
│   ├── handoff-format.instructions.md
│   ├── kb-format.instructions.md
│   └── notepad-format.instructions.md
├── agents/                        # Agent configurations
│   ├── research.agent.md
│   ├── writing.agent.md
│   └── review.agent.md
└── agent-state/                   # Runtime communication files
    ├── research-INBOX.md          # Messages for @research
    ├── research-HANDOFF.md        # Work assignments for @research
    ├── research-KB.md             # Persistent workflow lessons
    ├── research-NOTEPAD.md        # Temporary scratch space (cleared each session)
    ├── writing-INBOX.md
    ├── writing-HANDOFF.md
    ├── writing-KB.md
    ├── writing-NOTEPAD.md         # Temporary scratch space (cleared each session)
    ├── review-INBOX.md
    ├── review-HANDOFF.md
    ├── review-KB.md
    └── review-NOTEPAD.md          # Temporary scratch space (cleared each session)
```

## File Types Explained

### Communication Files (Per Agent)

**INBOX** - Asynchronous message queue
- Purpose: Inter-agent communication and updates
- Persistence: Messages move from New → Processed after handling
- Format: `.github/instructions/inbox-format.instructions.md`

**HANDOFF** - Active work assignment
- Purpose: Transfer work with full context and acceptance criteria
- Persistence: Only ONE active handoff per agent at a time
- Format: `.github/instructions/handoff-format.instructions.md`

**KB (Knowledge Base)** - Persistent workflow lessons
- Purpose: Store workflow improvements and user preferences ONLY
- Persistence: Permanent, never deleted (archive if obsolete)
- Contains: Collaboration patterns, quality standards, user constraints
- Does NOT contain: Project research findings, temporary notes, one-time decisions
- Format: `.github/instructions/kb-format.instructions.md`

**NOTEPAD** - Temporary scratch space
- Purpose: Organize thoughts during active work (task breakdown, drafts, observations)
- Persistence: Automatically cleared at the start of EVERY session
- Private: Never referenced by other agents
- Format: `.github/instructions/notepad-format.instructions.md`

## Collaboration Flow and Rules

Refer to `AGENTS.md` for end-to-end workflows, mandatory behaviors, acceptance criteria usage, and quality standards. This document intentionally avoids duplicating those details.

## Additional Documentation

- Behavioral rules and workflows: `AGENTS.md`
- File format specifications: `.github/instructions/*.instructions.md`
- Prompts and operational checklists: `.github/prompts/*`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/miguel-conde) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
