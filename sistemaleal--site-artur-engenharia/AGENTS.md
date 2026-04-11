# Copilot / AI agent instructions — Abeenengenharia site

This is a small, single-page static website (vanilla HTML/CSS/JS). The goal: be conservative when editing markup, preserve accessibility attributes, and follow the existing lightweight conventions.

- Big picture: `index.html` is the single entry. `script.js` implements behavior (WhatsApp links, mobile menu, and the portfolio carousel). `styles.css` contains all visual system tokens (CSS variables) and responsive rules.
- Data & assets: `assets/portfolio/portfolio.json` drives the carousel. Images live in `assets/portfolio/<category-id>/<filename>` and are referenced by the JSON `images` array.

- Key patterns to follow:
  - script.js is organized as small, self-contained blocks (WA link setup, menu, portfolio init). Edit in-place; prefer minimal changes and preserve ARIA attributes (e.g. `aria-*`, `role`, `tabindex`).
  - Portfolio: carousel reads `categories[]` from `portfolio.json`. Each category must have `id`, `label`, and `images[]`. Example entry:

```json
{
  "id": "my-category",
  "label": "My Category",
  "title": "My Title",
  "desc": "Short description",
  "images": ["01.png","02.png"]
}
```

  - When adding a category, create folder `assets/portfolio/<id>/` and add the image files named in `images[]`.
  - To add a new WhatsApp button: add the DOM element with an id (e.g. `myWhatsBtn`) and append its id to the `waIds` array in `script.js` (top of file). To change phone or base message, edit `const phone` / `baseMessage` in `script.js`.

- Developer workflows (project-specific):
  - No build step. Open `index.html` directly in a browser for static inspection, but to enable `fetch()` use a local static server (recommended):

```bash
python -m http.server 8000
# or
npx http-server .
```

  - There are no test or lint configs in the repo. Keep JS changes small and test in desktop + mobile widths (980px / 720px breakpoints used).

- Conventions and safety notes:
  - Keep CSS tokens in `:root` and reuse `.btn`, `.card`, etc. when styling new UI.
  - Preserve accessibility hooks: `aria-live`, `aria-controls`, `aria-expanded`, roles and keyboard handlers in the carousel/frame.
  - Avoid changing the WhatsApp phone number without explicit approval from the repo owner.

- Quick edits examples:
  - Add a portfolio image: add file to `assets/portfolio/<id>/` and push filename into the `images` array in `portfolio.json`.
  - Add menu item: insert anchor into the `.nav__menu` block in `index.html` and no JS changes required unless adding new section behavior.

- If merging into an existing `.github/copilot-instructions.md`: preserve any existing operational details and update the `Data & assets`, `Developer workflows`, and `Key patterns` sections with the latest notes above.

If any part of the site behavior is unclear (carousel error handling, phone number ownership, or intended deploy target), tell me which area to expand and I will refine this file.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Sistemaleal)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Sistemaleal)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
