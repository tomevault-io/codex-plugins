## Purpose
Short guidance for AI coding agents working on this repo: a tiny static Blackjack game implemented with `index.html`, `index.js`, and `style.css`.

## Big picture
- Single-page static web app (no build system). Serve by opening [index.html](index.html) in a browser.
- UI elements and game logic are tightly coupled: DOM IDs in [index.html](index.html) map directly to variables in [index.js](index.js).
- Core game flow lives in `index.js`: `startgame()` initializes `cardsa` and `sum`, `rendergame()` updates the DOM, `newcard()` draws additional cards.

## Key files
- [index.html](index.html): page structure and elements with IDs `a`, `b`, `c`, and `player_el` used by the script.
- [index.js](index.js): game state and functions. Important globals: `cardsa`, `sum`, `blackjak`, `isAlive`, `player`.
- [style.css](style.css): simple styling—no CSS framework or preprocessors.

## Project-specific conventions & patterns
- Minimal JS style: small number of global vars and plain functions (camelCase). Keep changes simple and imperative.
- DOM IDs are single letters (`a`, `b`, `c`)—do not rename IDs without updating both `index.html` and `index.js`.
- Random card logic: `randomCard()` returns `11` for Ace (1), `10` for face cards (>10), otherwise the numeric value. Preserve this when altering card logic.
- Inline event handlers are used (e.g., `onclick="startgame()"`). If converting to event listeners, update markup and ensure script still loads after DOM.

## Workflows (how to run and debug)
- Run locally: open [index.html](index.html) in any modern browser (DevTools for debugging).
- No build/test commands exist—avoid adding build tooling unless necessary.
- For live editing, use a simple static server (e.g., `npx serve .`), or VS Code Live Server extension.

## Safe change guidance (specific examples)
- Update UI text: change the message element by modifying `massag.textContent` in [index.js](index.js). Note variable `massage` in the file is the message text holder — be careful picking names.
- Update player chips example:

```js
// adjust chips and refresh display
player.chips -= 10
document.getElementById('player_el').textContent = player.name + ': $' + player.chips
```

- Add a new DOM element: add the element to [index.html](index.html) and reference it by id in [index.js](index.js); do not rely on querySelector ordering.

## Integration & external points
- No external dependencies or APIs. All logic is client-side.
- If adding libraries, include them via CDN or add a minimal package.json and document new developer steps.

## Merge guidance
- If `.github/copilot-instructions.md` already exists, merge by preserving any repo-specific rules and append concrete examples similar to the snippets above.
- Keep the file concise (20–50 lines) and focused on discoverable, actionable patterns.

## Questions for the maintainer
- Should DOM IDs be renamed to semantic names (e.g., `messageEl`, `cardsEl`)? I can update codebase and references if you want.
- Do you want a lightweight dev server + `package.json` added for local iteration?

If anything here is unclear or you'd like more examples (e.g., adding betting UI or refactoring to modules), tell me which area to expand.

---
> Source: [Zaherzi/Blackjack-Game](https://github.com/Zaherzi/Blackjack-Game) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-06 -->
