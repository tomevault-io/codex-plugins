# MeshcoreGRID Project Instructions

This repository extends MeshCore. Preserve MeshCore core behavior and data models unless a change is explicitly required.

MeshCore base code should remain untouched whenever possible so future upstream MeshCore updates can be integrated without unnecessary conflicts or regressions.

## Single source of truth rule

MeshCore is the single source of truth for all mesh/runtime behavior and persisted state.

- Do not mirror, cache, or shadow MeshCore contact/channel/node/routing state in GRID-owned models.
- Read and update authoritative state through existing MeshCore APIs and storage flows.
- The only allowed GRID-owned state is GUI-specific presentation state (for example: selection, temporary filters, scroll position, widget visibility, local animation state).
- GUI state must never become a parallel source of truth for MeshCore data.

## Core-first rule

For all feature work, fixes, UI integrations, storage changes, protocol handling, BLE behavior, channel management, time handling, and mesh behavior:

- Always prefer existing MeshCore core functions, structures, storage flows, and protocol semantics.
- Keep MeshCore base implementation untouched unless a change is strictly necessary and there is no safe extension point.
- Always check whether MeshCore already provides a function, slot model, persistence path, or protocol command before adding new logic.
- Do not create parallel GRID-only implementations when an existing MeshCore mechanism already exists.
- Do not duplicate or reinterpret MeshCore state in a separate custom model unless there is no safe core path available.
- When extending behavior, hook into the existing MeshCore flow instead of replacing it.
- Protect future upstream mergeability: prefer thin GRID layers over edits to MeshCore core files.

## Practical expectations

- Use existing channel APIs and storage slots instead of custom channel memory.
- Use existing BLE startup/control paths instead of inventing alternate transport state.
- Use existing MeshCore packet, contact, channel, and persistence semantics as the source of truth.
- Keep protocol compatibility with upstream MeshCore whenever possible.
- If a new GRID-specific layer is required, keep it UI-focused and thin over the MeshCore base.

## App structure rule

For GRID UI application modules (for example Messenger, Discover, MAP, Radio, Settings):

- Keep each app in its own dedicated source file to make iteration and maintenance easier.
- Prefer file names following the App_<Name> pattern (for example App_Map.cpp).
- Avoid embedding full app implementations into unrelated multi-app files, except for temporary scaffolding.
- Keep shared utilities in separate helpers and keep app files focused on that app's UI behavior.

## Decision rule

Before implementing a new feature, first answer:
1. Which existing MeshCore function or flow already does this?
2. Can GRID call or extend that directly?
3. Only if not, add the smallest possible project-specific layer.

When in doubt, choose the solution that keeps MeshCore base behavior most intact.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Quark1980)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Quark1980)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
