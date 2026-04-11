# Copilot Instructions — moslides

## Project Purpose

This is a **Node.js slide generation project** that programmatically builds a 50-slide Cloud Native 課程投影片 (master's-level course deck) using `pptxgenjs`. Each Part of the course (6 parts total) is a separate script that produces its own `.pptx` file, which is then merged into a final deck.

## Running Scripts

```bash
# Generate slides for a specific part
node src/part1.js         # outputs output/part1.pptx
node src/part2.js         # outputs output/part2.pptx
# ... and so on for part3–part6

# Merge all parts into final deck
node src/merge.js         # outputs output/cloud_native_slides_redesigned.pptx
```

No test suite exists. Visual QA is done by converting PPTX to images:

```bash
# Convert a PPTX to PDF, then to JPEG images for QA
mkdir -p output/qa/partN
python3 /Users/hwchiu/.claude/plugins/cache/anthropic-agent-skills/document-skills/b0cbd3df1533/skills/pptx/scripts/office/soffice.py \
  --headless --convert-to pdf output/partN.pptx --outdir output/qa/partN/
pdftoppm -jpeg -r 150 output/qa/partN/partN.pdf output/qa/partN/slide
# Produces: slide-01.jpg, slide-02.jpg, ...
```

## Architecture

```
src/
  design-system.js   # All color + font constants (single source of truth)
  icon-helper.js     # react-icons → PNG base64 via ReactDOMServer + sharp
  part1.js           # Slides 1–12  (Part 1: 傳統部署演進)
  part2.js           # Slides 13–20 (Part 2: Scale Out 挑戰)
  part3.js           # Slides 21–26 (Part 3: Container 革命)
  part4.js           # Slides 27–34 (Part 4: 12-Factor App)
  part5.js           # Slides 35–42 (Part 5: DevOps 整合)
  part6.js           # Slides 43–50 (Part 6: SDLC 閉環)
  merge.js           # Merges all 6 part PPTX into one final file
output/              # Generated PPTX files + QA images (not committed except final)
docs/plans/          # Design spec and implementation plan (reference docs)
```

Each `partN.js` script is standalone: it requires `design-system.js` and `icon-helper.js`, imports icons from `react-icons/fa`, defines its own local helper functions, and ends with an `async main()` that builds and saves the PPTX.

## Key Conventions

### Design System — always use `src/design-system.js`

```js
const { COLORS, FONTS } = require("./design-system");
```

- **Colors are hex strings WITHOUT `#` prefix** (pptxgenjs format): `COLORS.accent` → `"58A6FF"` not `"#58A6FF"`
- Never hardcode colors inline — always reference `COLORS.*`

### Component Color Assignments (enforced across all slides)

| Component | Color constant | Hex |
|-----------|---------------|-----|
| Frontend / Nginx | `COLORS.frontend` | `1F6FEB` (blue) |
| Backend / App Server | `COLORS.backend` | `238636` (green) |
| Database | `COLORS.database` | `E36209` (orange) |
| LB / Cache / MQ (infra) | `COLORS.infra` | `6E40C9` (purple) |
| Container / Pod | `COLORS.container` | `0D8A6C` (teal) |
| Client / Browser | `COLORS.client` | `8B949E` (grey) |

### Icon Usage

Icons come from `react-icons/fa` and are rendered to PNG base64 via `iconToBase64()`:

```js
const { FaServer, FaDatabase } = require("react-icons/fa");
const { iconToBase64 } = require("./icon-helper");

// In async context:
const serverIcon = await iconToBase64(FaServer, "#" + COLORS.backend, 256);
slide.addImage({ data: serverIcon, x: 1, y: 1, w: 0.5, h: 0.5 });
```

Note: `iconToBase64` returns `"image/png;base64,..."` — pass directly to `slide.addImage({ data: ... })`.

### Slide Layout

- Canvas: **10" × 5.5"** (pptxgenjs widescreen default)
- Coordinates are in inches; all `x`, `y`, `w`, `h` values are floats
- Background always set as: `slide.background = { color: COLORS.bg };`

### Per-Part Helper Pattern

Each `partN.js` defines local helpers at the top (not shared modules), following the pattern established in `part1.js`:

- `addPartLabel(slide, text)` — top-right part indicator
- `addComplexityBar(slide, pres, current, total, x, y)` — visual complexity progress bar
- `addTip(slide, pres, text, y)` — 💡 tip card with accent border
- `addCard(slide, pres, { x, y, w, h, fill, lineColor, lineWidth })` — rounded rect card with shadow
- `addArrow(slide, x, y, label)` — directional arrow label

Copy these helpers into each new part file — they are intentionally duplicated per file, not extracted into a shared module.

### Shape Types

Use `pres.ShapeType.*` for shapes (e.g., `pres.ShapeType.rect`, `pres.ShapeType.roundRect`). The `pres` object must be passed into slide functions for shape access.

### Slide Template Types (from design spec)

| Template | Use case |
|----------|----------|
| T1 | Cover page |
| T2 | Part opener (large part number, bullet list) |
| T3 | Architecture evolution (diagram top 65%, pro/con cards bottom 35%) |
| T4 | Comparison (❌ left / ✅ right, dual column) |
| T5 | Concept explanation (large icon left, 3–4 bullet points right) |
| T6 | Code page (dark code card + diagram) |
| T7 | Part summary (2×3 card grid) |

### Commit Convention

```
feat: add Part N slides (description in English)
```

### QA Checklist (before committing each part)

- All architecture diagrams use icons, not ASCII arrows
- Component colors match the design system table above
- Complexity bar progresses correctly (Part 1 low → Part 2 high → Part 3 drops → Part 4–6 stable low)
- No text overflow or element overlap
- Chinese text displays correctly (font: `Calibri`, code: `Consolas`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hwchiu)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/hwchiu)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
