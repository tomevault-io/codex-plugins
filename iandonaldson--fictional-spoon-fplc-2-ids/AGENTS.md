# DOE Architecture AI Agent Instructions

This project implements a 3-layer Directive-Orchestration-Execution (DOE) architecture to separate probabilistic AI decision-making from deterministic business logic.

## Architecture Overview

**Layer 1: Directives** (`directives/`)
- Markdown SOPs defining goals, inputs, tools, outputs, and edge cases
- Living documents that improve through experience
- Natural language instructions for workflows

**Layer 2: Orchestration** (You, the AI agent)
- Intelligent routing between directives and execution tools
- Handle errors, ask for clarification, update directives with learnings
- DO NOT implement logic yourself—call execution scripts instead

**Layer 3: Execution** (`execution/`)
- Deterministic Python scripts handling APIs, data processing, file operations
- Use `.env` for credentials and environment variables
- Must be reliable, testable, and well-commented

## Critical Workflows

**Before writing any script:**
1. Check `execution/` directory for existing tools per the directive
2. Only create new scripts if none exist for the task

**Path Handling (CRITICAL):**
- **ALWAYS use absolute paths** when calling scripts or referencing data files
- Workspace root: `/workspaces/fictional-spoon-fplc-2-ids`
- Terminal working directory is unreliable—never assume `pwd`
- In Python scripts, calculate absolute paths: `Path(__file__).parent.parent / "data" / "file.txt"`
- In terminal commands, use full paths: `/workspaces/fictional-spoon-fplc-2-ids/execution/script.py`
- This prevents "file not found" errors from wrong working directories

**API Verification (CRITICAL):**
- **Documentation can be outdated**—verify actual installed APIs before writing code
- When imports fail, check what's available: `python -c "import module; print(dir(module))"`
- Test API calls with small snippets before writing full scripts
- If API differs from docs, document the actual API in `directives/` for future reference
- Reality beats documentation—use what actually works, not what should work

**Self-annealing loop (when errors occur):**
1. Read error message and stack trace
2. Fix the script and test it (ask user first if it uses paid API credits)
3. Update the relevant directive with what you learned
4. System is now stronger

**Example:** Hit API rate limit → investigate API docs → find batch endpoint → rewrite script → test → update directive with rate limit constraints.

## File Organization

**Directory structure:**
- `.tmp/` - Intermediate files (dossiers, scraped data, temp exports). Never commit, always regenerable
- `execution/` - Python scripts (deterministic tools)
- `directives/` - Markdown SOPs (instruction set)
- `.env` - Environment variables and API keys (gitignored)
- `credentials.json`, `token.json` - Google OAuth (gitignored)

**Key principle:** Local files are for processing only. Deliverables live in cloud services (Google Sheets, Slides, etc.) where users can access them. Everything in `.tmp/` can be deleted and regenerated.

## Directive Management

- Directives are your instruction set—preserve and improve them over time
- Update directives when you discover API constraints, better approaches, common errors, timing expectations
- DO NOT create or overwrite directives without asking unless explicitly instructed
- Never extemporaneously use and discard directive patterns

## Operating Principle

You make decisions, scripts do work. 90% accuracy per step = 59% success over 5 steps. Push complexity into deterministic code so you focus on intelligent routing.

Reference: [copilot.md](../copilot.md) for comprehensive architecture explanation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/iandonaldson)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/iandonaldson)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
