# NovelTools — Copilot Instructions

This workspace uses **NovelTools**, a VS Code extension for novel writing. The manuscript is organized as individual `.md` scene files in chapter folders, with structure defined in `noveltools.json`.

## Scene Statuses

Scenes have 6 statuses stored in `noveltools.json` → `sceneStatus` (keyed by relative path with forward slashes):

- `drafted` (🟡) — first draft written
- `revision` (🔵) — undergoing revision
- `review` (🟠) — ready for or under review
- `done` (🟢) — final, complete
- `spiked` (🔴) — temporarily removed
- `cut` (⚫) — permanently removed

To change a scene's status, edit the `sceneStatus` object in `noveltools.json`:

```json
{ "sceneStatus": { "chapter-01/opening.md": "revision" } }
```

## Scene Metadata

Per-scene metadata lives in `noveltools.json` → `sceneMetadata`:

```json
{ "sceneMetadata": { "chapter-01/opening.md": { "synopsis": "Short description." } } }
```

## Project Schema

The JSON schema at `schemas/noveltools-project.schema.json` provides validation and autocomplete for `noveltools.json`.

---
> Source: [Arcadesys/NovelTools](https://github.com/Arcadesys/NovelTools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-29 -->
