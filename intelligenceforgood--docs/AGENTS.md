# Copilot Instructions for i4g/docs

**Unified Workspace Context:** This repository is part of the `i4g` multi-root workspace. Shared coding standards, routines, and platform context live in the `copilot/` repo. These instructions contain only repo-specific context.

## Purpose

This is the **GitBook documentation site** for end users of the I4G Platform — analysts, administrators, and law enforcement partners. It is NOT a developer documentation repo.

## Critical Rule: End-User Content Only

**Do NOT add developer-facing documentation to this repo.** This includes:

- Architecture internals, API implementation details, infrastructure setup
- Developer onboarding, Copilot workflow guides, coding standards
- Debugging guides, deployment procedures, config manifests

Developer documentation belongs in:

- `copilot/docs/` — Copilot workflow guides and developer onboarding
- `core/docs/` — API design notes, architecture internals
- `infra/docs/` — Infrastructure setup

## Structure

```
docs/book/
├── SUMMARY.md              # Table of contents — MUST be updated when adding pages
├── guides/                 # End-user guides (analyst, admin, law enforcement)
├── api/                    # API reference for integrators
├── architecture/           # High-level architecture for technical end users
├── ssi/                    # SSI user guide
└── security/               # Security and compliance documentation
```

## Adding Pages

1. Create the Markdown file in the correct `book/` subdirectory.
2. Add an entry to `book/SUMMARY.md` — pages not in SUMMARY will not appear in the site.
3. Use present tense, active voice, second person. Lines ≤ 120 chars.
4. Code snippets: focused excerpts (3–15 lines) only; link to full file paths rather than pasting entire files.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/intelligenceforgood)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/intelligenceforgood)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
