# KrakenD Operator AI Development Instructions

## CRITICAL: Read Project State First

### At Session Start Before making ANY changes, read [.github/PROJECT_STATE.md](.github/PROJECT_STATE.md).
- Understand current state, recent changes, and pending work
- Review any in-progress items or architectural decisions
- Current implementation status and architecture
- Domain-specific patterns (shadow mode, correlation ID flow, slip resolution)
- Code examples and anti-patterns
- Environment variable reference

---

## Required Reading

1. **[.github/PROJECT_STATE.md](.github/PROJECT_STATE.md)** - Project context, architecture, domain patterns
2. **[.github/instructions/go.instructions.md](.github/instructions/go.instructions.md)** - Go coding standards

---

## Core Rules

### Research First, Never Assume
- Check existing code and tests before implementing
- Use Context7 MCP for third-party library documentation
- Never guess at function signatures or import paths

### No Incomplete Code
- **NEVER** leave TODO stubs, placeholder implementations, or "not yet implemented" code
- Every function must be fully implemented before moving on

### Dependency Injection
- Accept interfaces as parameters, return concrete types
- All external dependencies must be injectable for testing

### Test-Driven Development
Follow TDD strictly: **Red → Green → Refactor**

### Code Quality
- All code must pass `golangci-lint` with zero errors
- No `//nolint` directives without explicit user approval
- Minimum 80% test coverage required

---

## Validation (Required Before Completion)

```bash
go fmt ./...
go mod tidy
golangci-lint run -c .github/.golangci.yml          # Zero errors required
go test -v -race -coverprofile=coverage.out ./...   # 85%+ coverage required
go build -o KrakenD-Operator ./operator/cmd/KrakenD-Operator
```

---

## Project State Documentation

**You MUST maintain `.github/PROJECT_STATE.md` as working memory for this project.**

### At Session Start
- **Read `.github/PROJECT_STATE.md`** to understand current state, recent changes, and pending work
- Review any in-progress items or architectural decisions

### During Work
Update PROJECT_STATE.md whenever:
- Implementing new features or systems
- Making architectural decisions or changes
- Discovering technical debt or issues
- Completing significant milestones
- User provides direction that affects project structure

### Required Sections
```markdown
# Project State — KrakenD Operator

> **Last Updated:** [Date]
> **Status:** [Brief current state summary]

## Overview
[What this application does, key characteristics]

## Implemented Systems
[Breakdown of completed functionality by component]

## Recent Changes
[What changed in recent sessions, new components/systems/patterns]

## Current Focus
[What's being worked on now, immediate next steps]

## Architectural Decisions
[Key design choices made, rationale, tradeoffs]

## Technical Debt / Known Issues
[Outstanding problems, things to fix later]

## Next Steps (Not Yet Implemented)
[Planned features, improvements, user requests pending]
```

### Update Discipline
- Keep entries **concise but specific** (reference file paths, function names)
- **Date entries** in Recent Changes section
- Remove outdated items from "Next Steps" as they're completed
- If PROJECT_STATE.md doesn't exist, **create it** with current project state

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MyCarrier-DevOps)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/MyCarrier-DevOps)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
