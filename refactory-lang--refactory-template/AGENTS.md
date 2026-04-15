# Copilot Instructions

## Spec-Driven Development

This repository uses **speckit** for specification-driven development. All features begin as specifications before implementation.

## When Assigned an Issue Labeled `needs-spec`

When you are assigned an issue with the `needs-spec` label, your job is to create a formal specification from the issue content:

1. **Read the issue body** carefully. Look for:
   - A `## To create the spec` section with a suggested `/speckit.specify` command
   - `## Overview` / `## Scope` / `## Success Criteria` sections with context
   - `## Context` / `## Dependencies` for dependency information

2. **Create the feature branch and spec** by running the speckit create-new-feature script:
   ```bash
   .specify/scripts/bash/create-new-feature.sh "" --json --short-name "<short-name>" "<feature description from issue>"
   ```
   - Extract a 2-4 word short name from the issue title (e.g., "tier2-control-flow")
   - Do NOT pass `--number` — the script auto-detects the next available number
   - The script creates the branch and initializes the spec directory

3. **Write the specification** to the spec file path returned by the script (in `specs/<branch>/spec.md`). Follow the template at `.specify/templates/spec-template.md`. The spec must:
   - Focus on WHAT and WHY, not HOW (no implementation details)
   - Include user stories, functional requirements, and success criteria
   - Reference the source issue URL in the spec
   - Include any dependencies mentioned in the issue

4. **Create the quality checklist** at `specs/<branch>/checklists/requirements.md` and validate the spec against it.

5. **Comment on the issue** with the spec location:
   ```
   Specification created: `specs/<branch>/spec.md` on branch `<branch>`
   ```

6. **Update issue labels**: remove `needs-spec` and `spec-ready`, add `has-spec`.

## When Assigned Other Issues

For issues without `needs-spec`, follow standard development practices:
- Read CLAUDE.md and AGENTS.md for project context
- Follow existing code patterns and conventions
- Write tests for new functionality

## Project Structure

- `specs/` — Feature specifications (spec.md, plan.md, checklists/)
- `.specify/` — Speckit configuration, templates, scripts, extensions
- `.specify/templates/` — Templates for specs, plans, tasks, checklists
- `.specify/scripts/bash/` — Automation scripts (create-new-feature.sh, setup-plan.sh)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/refactory-lang) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
