# Agent Workflow Guide

This document defines how coding agents (like GitHub Copilot, Cursor, or other AI assistants) should work in this repository.

## ExecPlans

When making **complex changes** to this codebase, write and follow an **ExecPlan** as defined in `.agent/PLANS.md`.

### When to Create an ExecPlan

Create an ExecPlan for:

- **New Worker endpoints** (adding routes, handlers, validation logic)
- **Schema changes** (modifications to `gpt/openapi.yaml` or KBM structure)
- **Multi-component changes** (changes spanning worker/ + gpt/ + notebooks/)
- **Security or performance work** (sanitization, validation, rate limiting)
- **Refactoring** (restructuring code while maintaining behavior)
- **Test infrastructure** (adding test frameworks, significant test coverage)

### When an ExecPlan is Optional

Skip ExecPlans for:

- Typo fixes and documentation updates (single file, obvious)
- Simple bug fixes (single function, clear root cause)
- Dependency updates (version bumps with no API changes)
- Example additions (new files in `examples/`)

## How to Use ExecPlans

1. **Before coding**: Create an ExecPlan file in `.agent/plans/` using the format from `.agent/PLANS.md`
2. **During implementation**: Update the Progress section as you complete milestones
3. **Track decisions**: Document why you chose specific approaches in the Decision Log
4. **Record discoveries**: Note unexpected issues or insights in Discoveries
5. **After completion**: Add a brief Retrospective summarizing what worked and what didn't

## The ExecPlan is the Ground Truth

When reviewing PRs that include an ExecPlan:

- The plan should be self-contained (a developer can implement from the plan + repo alone)
- The diff should match the plan's milestones
- Acceptance criteria should be demonstrably satisfied
- Progress updates should reflect actual implementation state

## Repository-Specific Patterns

### Worker Changes

For changes to `worker/src/index.ts`:

- Include curl commands showing request/response examples
- Document expected HTTP status codes and headers
- Show how to verify the change locally (`wrangler dev`)
- Update `gpt/openapi.yaml` if the API contract changes

### GPT Integration Changes

For changes to Custom GPT resources:

- Keep `gpt/openapi.yaml` and Worker implementation synchronized
- Update `gpt/CUSTOM_GPT_INSTRUCTIONS.md` if behavior changes
- Test with example requests from the OpenAPI schema

### Notebook Changes

For changes to Colab notebooks:

- Document external dependencies (GH_TOKEN, Drive access)
- Include example outputs or screenshots
- Note any API rate limit considerations

## See Also

- `.agent/PLANS.md` - Detailed ExecPlan format specification
- `.agent/examples/` - Example ExecPlans for reference
- `CONTRIBUTING.md` - General contribution guidelines
- `docs/ARCHITECTURE.md` - System architecture overview

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CyberBASSLord-666) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-13 -->
