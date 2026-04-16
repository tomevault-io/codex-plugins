# Copilot Instructions for PortOSC

## Project Snapshot
- Tech: `.NET 8`, `Windows Forms` (targeting Visual Studio Community 2026)
- Main goal: serial/network debugging + lightweight oscilloscope plotting
- Current architecture: event-driven, but `Form1` is highly coupled to transport and parsing logic

## What to optimize first
1. Reduce duplication in `Form1` state/UI toggling logic
2. Extract parsing/forwarding logic from UI handlers
3. Keep behavior compatible while refactoring
4. Address root causes of issues rather than superficial fixes; prefer deeper behavioral correctness during refactoring

## Coding Rules
- Prefer behavior-preserving changes and follow the current task's refactor scope
- Do not introduce new dependencies unless necessary
- Reuse existing utilities in `src/Tools.cs` when possible
- Keep transport code (`Serial/TCP/UDP`) independent from UI controls
- Use async APIs consistently; avoid fire-and-forget when reliability matters
- Implement large-scale refactoring in a single pass rather than breaking it into smaller steps; continue through multiple compile-verified substeps without pausing for each small stage, and only stop for necessary runtime validation or after a substantial batch of refactoring
- After each modification, perform local Git commits without pushing
- Maintain version control using the current milestone scheme:
  - `1.0.x`: historical local revisions
  - `1.1.0`: first push-ready milestone
  - `1.1.x`: local revisions after the push-ready milestone
- If asked to modify the current commit, amend it instead of creating a new commit

## File/Layer Expectations
- UI orchestration: `forms/`
- Transport and services: `src/`
- Reusable controls/widgets: `widget/`
- Docs and architecture notes: `docs/`

## Validation
- After changes, run build verification
- If changing parsing behavior, document input/output examples

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KeliWontKeil) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
