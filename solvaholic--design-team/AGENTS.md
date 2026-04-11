# Design Team Repository Instructions

This repository implements an AI-powered design thinking workflow system with specialized agents that guide projects through research, ideation, and validation phases.

## Repository Structure

This is a **design system workspace** containing:
- **Agent definitions** in `.github/agents/` — AI agent personas (Stakeholders, DesignTeam, LT)
- **Design skills** in `.github/skills/` — Reusable design thinking methods
- **Project templates** in `templates/` — Structures for new design projects
- **Data schemas** in `schemas/` — JSON schemas for project state
- **Active projects** in `projects/` (created by users) — Actual design work

## Core Concepts

### Design Thinking Phases
Projects move through: **Empathize → Define → Ideate → Prototype → Iterate**

Each phase has specific activities, artifacts, and decision gates.

### Agents Available
- **@DesignTeam** — Five designers guiding the design process (use for most work)
- **@Stakeholders** — User representatives providing authentic feedback
- **@LT** — Leadership making strategic decisions at phase gates

See `.github/agents/README.md` for full agent documentation.

### Skills Framework
Agents apply reusable skills from `.github/skills/`:
- empathy-mapping, needs-articulation
- assumption-grading, assumption-validation  
- insight-synthesis, idea-evaluation
- prototype-planning, playback-preparation

### Custom Tool System
This repository uses custom Python tools via `scripts/tool_runner.py` for all file operations and state management. Agents access these tools using the `execute` tool to run Python scripts.

See [Custom Tools Documentation](../docs/CUSTOM_TOOLS.md) for:
- Tool architecture and usage patterns
- Complete list of read and edit operations
- Standardized JSON response format
- Integration with agents

## Working with Projects

### Project State
Every project has `projects/[name]/currentstate.json` as the central index of:
- Current phase
- Assumptions with certainty/risk grades
- Stakeholders with needs/pain points
- Research plan and insights
- Ideas with impact/feasibility grades
- Playback decisions

**Always read currentstate.json first** to understand project context.

### Project Artifacts
- `insights/` — Research notes, observations, interviews, synthesis
- `ideas/` — Solution concepts and prototype iterations
- `playbacks/` — Leadership presentations and decisions

## General Guidance

When working in this repository:
- Focus on user needs, not features
- Validate assumptions before commitments
- Document reasoning and evidence
- Use the design thinking phases systematically
- Keep currentstate.json up to date

For specific file types, see path-specific instructions in `.github/instructions/`.

## Documentation

- [Repository instructions guide](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/add-repository-instructions)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/solvaholic)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/solvaholic)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
