# Potato — GitHub Copilot instructions

GitHub Copilot in VS Code auto-loads this file as system context for every
session in this repo. Keep it short, accurate, and aligned with what's
actually on disk.

## What this project is

**Potato** is a single-file architecture diagramming tool — `index.html` is
the entire editor (~8700 lines, vanilla JS + HTML + CSS, zero dependencies,
zero build step).

Diagrams are saved as self-contained `.potato.html` files with the diagram
JSON embedded in a `<script type="application/json" id="potato-data">` block,
plus an interactive viewer (hover tooltips, ▶ Play Flow, theme toggle).

## Hard rules

1. **No npm packages at runtime.** Browser code in `index.html` stays vanilla
   ES6+, HTML, and CSS. No React, no TypeScript, no JSX, no import/export.
2. **No bundler.** Plain `<script>` only. The editor must work from `file://`
   by double-clicking the HTML — so no `fetch()` of local files.
3. **All editor code lives in `index.html`.** Don't split it into modules
   unless explicitly asked.
4. **CSS uses `var(--variable)`.** Extend via `:root` (and `[data-theme="light"]`
   for the light variant). Never hardcode colors.
5. **System fonts only.** No Google Fonts, no CDN. Offline-first is a
   first-class promise.
6. **User-controlled strings → `escHtml()`** before they hit `innerHTML` or
   an SVG attribute. Labels, descriptions, narration text, AI imports, all of it.

## State model — four arrays

```js
let nodes    = [];  // { id, x, y, w, h, type, label, sublabel, category, icon, iconUrl, theme, description, zIndex }
let arrows   = [];  // { id, from, to, fromPort, toPort, color, style, label, animated }
let groups   = [];  // { id, x, y, w, h, label, color }
let playFlow = [];  // [{ arrow: arrowId, text: "User opens app..." }, ...]
```

Always mutate these in place, then re-render. The save format mirrors them
exactly, plus a `meta` object (`version`, `name`, `created`, `updated`).

## Required calls after state changes

When writing new editor features that mutate state, follow the existing discipline:

- `markDirty()` — after any mutation
- `pushHistory()` — **before** destructive ops (enables undo/redo)
- `redrawArrows()` — after moving or resizing nodes
- `updateStatusBar()` / `updateMinimap()` — after adding or removing nodes
- `notify(msg, type)` — for user feedback. Never fail silently. `type` is `'info'` (default), `'warning'`, or `'error'`.

## Valid enum values

```
node.theme:    green | orange | blue | purple | red | teal | pink | yellow | cyan | gray | gradient
arrow.color:   default | green | blue | purple | red | teal | orange | pink | yellow
arrow.style:   solid | dashed | dotted
fromPort/toPort: top | bottom | left | right
group.color:   purple | blue | green | orange | red | teal | pink
```

The AI Import path validates these and sanitises invalid values; new code
should validate at the boundary too.

## Key functions

```js
// Rendering
renderNode(node)         // create/update node DOM
renderGroup(grp)         // create/update group DOM
drawArrow(arrow)         // draw orthogonal SVG path
redrawArrows()           // redraw all arrows
fitToContent()           // zoom/pan to fit

// State
addNodeFromComp(comp,x,y)// drop from sidebar; auto-fills SERVICE_KB data
loadDiagramData(data)    // load { nodes, arrows, groups, playFlow, meta }
getDiagramData()         // current snapshot, preserves meta.created
pushHistory() / undo() / redo()   // 60-level history

// Save / share
saveDiagram()            // Ctrl+S — saves the current diagram (download / FileSystem handle)
saveAsDiagram()          // Ctrl+Shift+S — prompts for new path
buildSavedHTML(name, dataStr, svgMarkup)   // assembles the standalone viewer

// Import
showPasteModal() / importFromAI()  // LLM HTML or raw JSON
parseMermaid(src)                  // Mermaid → Potato data

// UI helpers
notify(msg, type)        // toast: 'info' | 'warning' | 'error'
escHtml(s)               // HTML-escape user strings (also escapes ' and handles 0/false)
inferKbForNode(node)     // pull SLA/limits/cost/pitfalls/whenToUse from SERVICE_KB
```

## Service Knowledge Base

The KB lives in [`service_kb.json`](../service_kb.json) and is spliced into
`index.html` by `_generate_kb.js`. Entry shape:

```json
{
  "lambda": {
    "name":         "AWS Lambda",
    "matches":      ["lambda", "AWS Lambda"],
    "sla":          "99.95% monthly uptime",
    "limits":       "Max execution: 15 min | Memory: 128 MB–10 GB | ...",
    "whenToUse":    "Short-lived, event-driven tasks. Scales to zero.",
    "cost":         "$0.20/M requests + $0.0000166/GB-s. Free tier: 1M req...",
    "costEstimate": { "small": 5, "medium": 200, "large": 2400 },
    "pitfalls":     ["VPC cold starts add 1–5s", "..."]
  }
}
```

Currently 52 services documented, 39 priced (the 13 unpriced are Azure / GCP / generic shapes). After editing the JSON, run:

```bash
node _generate_kb.js
```

## Icons

1067 SVGs under `icons/<provider>/<category>/`. The catalog and inlined SVG
text are both spliced into `index.html` by `_generate_components.js`.

After adding/removing/renaming any SVG:

```bash
node _generate_components.js
```

CI fails if these generators would produce a diff against the committed
`index.html` — see `.github/workflows/check.yml`.

## Saved file format

```html
<!DOCTYPE html>
<html lang="en">
<head>...</head>
<body>
<script type="application/json" id="potato-data">
{
  "meta":     { "version": "1.0", "name": "...", "created": "ISO", "updated": "ISO" },
  "nodes":    [...],
  "arrows":   [...],
  "groups":   [...],
  "playFlow": [...]
}
</script>
<script type="application/json" id="potato-viewer-payload">
{ "kb": { /* per-node KB snapshot */ }, "asOf": "2026-Q2" }
</script>
<!-- inline viewer SVG + script for hover tooltips + Play Flow + theme toggle -->
</body>
</html>
```

The parser looks for `id="potato-data"` and `id="potato-viewer-payload"` —
never change these identifiers without bumping `meta.version` and updating
the loader.

## Security

User-controlled strings flow into the DOM via `innerHTML` in several places.
Always wrap with `escHtml()`. When emitting JSON into a `<script>` island,
also escape `</script` sequences:

```js
JSON.stringify(data, null, 2).replace(/<\/scr(ipt)/gi, '<\\/scr$1');
```

Without that, a label like `</script><script>alert(1)</script>` will break
the saved file's JSON island.

## Tech stack

- Pure vanilla JS, HTML5 Canvas (minimap), SVG (arrows + saved viewer)
- CSS variables for theming (dark + light)
- localStorage (autosave), sessionStorage (per-tab id), IndexedDB
  (FileSystemFileHandle persistence on Chromium), BroadcastChannel
  (cross-tab save notifications)
- Pointer Events — mouse, trackpad, touch, pen all use the same code paths
- FileReader (open), Blob/URL (download), HTML5 Drag & Drop (sidebar)

## What Potato is NOT

- Not a React/Vue/Angular app
- Not a Node.js server
- Not an Electron app (yet — on roadmap)
- Not SaaS — no accounts, no telemetry, no cloud
- Not a generic whiteboard — the cloud-icon-heavy focus is intentional

---
> Source: [abhisheksingh011/potato](https://github.com/abhisheksingh011/potato) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-26 -->
