# Copilot Instructions for Matrix-p5

## Project Overview

Matrix-p5 is a Matrix digital rain animation built with [p5.js](https://p5js.org/). It renders falling Katakana character streams in a classic green-on-black aesthetic, deployed as a GitHub Pages site.

## Behavioral Guidelines

### Matrix Aesthetic and Terminology

When appropriate, use Matrix-themed language in documentation, commit messages, and comments:

- **"Wake up, Neo..."** — for initialization messages and setup routines
- **"Follow the white rabbit"** — for optimization paths and performance improvements
- **"There is no spoon"** — for explaining abstraction tricks or indirect rendering techniques
- **"Free your mind"** — for unconventional solutions or creative approaches
- **"The red pill reveals truth"** — for debug mode, diagnostics, and verbose logging

**Balance:** Use Matrix references naturally in documentation and comments, but prioritize clarity over theme. Code comments should be professional and beginner-friendly. Never sacrifice readability for style.

### Code Style

- Use `let` and `const` in preference to `var`.
- Keep functions small and focused; prefer descriptive names over abbreviations.
- Comment complex logic, especially math-heavy rendering code, so beginners can follow along.
- Avoid breaking the existing p5.js setup/draw loop pattern unless there is a compelling reason.

### p5.js Conventions

- Canvas is created in `setup()` with `createCanvas(windowWidth, windowHeight)`.
- Handle window resize in `windowResized()` by calling `resizeCanvas()` and rebuilding state.
- Use an off-screen `createGraphics()` buffer (`gfx`) for rendering streams, then composite with `image(gfx, 0, 0)` in `draw()`.

### Performance

- Minimize per-frame object allocation — reuse `Stream` instances where possible.
- The optional post-processing blur (`gfx.filter(BLUR, 5)`) is disabled by default due to cost; keep it commented out unless the user explicitly enables it.

### Accessibility & UX

- The page should always render full-screen with no scrollbars, margins, or visible browser chrome.
- On mobile, ensure `width: 100%; height: 100%` on `html` and `body` so the canvas fills the viewport.
- The Screen Wake Lock API is used to keep the display on during the animation; handle permission errors silently.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ap0ught) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
