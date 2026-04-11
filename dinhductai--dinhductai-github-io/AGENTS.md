# Copilot Instructions - Terminal Portfolio

## Project Overview
This is a **terminal-themed portfolio website** built with vanilla HTML/CSS/JavaScript (no frameworks). The site mimics a Linux terminal interface where users type commands to navigate content.

**Tech Stack:** Pure vanilla JS, CSS3 custom properties, Google Fonts (Fira Code)
**Deployment:** GitHub Pages (repository name: `dinhductai.github.io`)

---

## Architecture & Key Components

### 1. Command System (`script.js`)
- **Central Data Structure:** `commands` object (lines 18-104) maps command names to HTML content
- **Help Menu:** Defined separately as `helpMenu` (lines 16-26) for maintainability
- **Command Processing Flow:**
  1. User types → `hiddenInput` captures (mobile-friendly)
  2. `executeCommand()` validates → `processCommand()` displays output
  3. Output injected directly as HTML into `#output` div

**Adding New Commands:**
```javascript
// In the commands object (script.js ~line 28):
newcommand: `
<span style="color: var(--blue);">Title</span>
Content with <strong>HTML</strong> formatting
`
```
Then add to `helpMenu` at line ~20.

### 2. Terminal UI Structure (`index.html`)
```
#terminal (container)
├── #header (macOS-style window buttons)
├── #output (scrollable content area)
└── #input-line (prompt + visible input + caret)
    └── #hidden-input (actual input field, invisible)
```

**Why Hidden Input?** 
- Enables mobile keyboard without breaking desktop UX
- Position: absolute, opacity: 0 (style.css line 149)
- Font-size: 16px prevents iOS zoom on focus

### 3. Styling System (`style.css`)
**CSS Variables (lines 1-9):** Color scheme based on VS Code Dark+ theme
- `--green`: #00ff7f (prompts, highlights)
- `--blue`: #569cd6 (links, tech keywords)
- `--red`: #f44747 (errors)

**Critical Responsive Breakpoint:** 768px (line 160)
- Mobile: Full viewport height (100vh), no border-radius
- Desktop: Fixed 600px height with scroll

---

## Developer Workflows

### Local Development
```bash
# No build process - just open index.html in browser
# Or use simple HTTP server:
npx http-server -p 8080
```

### Testing Checklist
1. **Desktop:** Click terminal → type commands → verify scroll behavior
2. **Mobile:** 
   - Tap terminal → keyboard appears
   - Type → `#input` updates in real-time
   - Submit → content scrolls to bottom (setTimeout fix at line 138)
3. **Command History:** ↑↓ arrows cycle through previous commands

### Deployment
Push to `main` branch → GitHub Pages auto-deploys from root directory.

---

## Project-Specific Conventions

### Content Formatting Patterns
**From `commands` object examples:**
- Section headers: `<span style="color: var(--green);">Title</span>`
- Technical terms: `<span style="color: var(--blue);">Java 21</span>`
- Links: Use `target="_blank"` for external URLs
- Multi-line: Use template literals with proper indentation

### Scroll Behavior Fix
**Critical Pattern (lines 135-138):**
```javascript
setTimeout(scrollToBottom, 10);
```
Why? Ensures DOM renders before scrolling. Without delay, scroll calculates wrong height.

### ASCII Art Handling
- Use `<pre>` tags with `font-family: 'Fira Code'`
- Set `overflow-x: auto` for horizontal scroll on mobile
- Keep line length < 60 chars for mobile readability (see `arch` command)

---

## Common Modifications

### Update Portfolio Content
1. Edit `commands` object in `script.js`
2. Maintain color coding: green=section, blue=tech, yellow=new
3. Test on mobile: ensure no horizontal scroll except in `<pre>` blocks

### Add New Sections
1. Add command to `commands` object
2. Update `helpMenu` string (maintain color-coded format)
3. Verify `clear` command still resets to welcome message

### Styling Changes
- Modify CSS variables (style.css line 1-9) to change entire color scheme
- Adjust `#terminal` max-width (line 30) for different desktop sizes
- Terminal height: line 32 (desktop), line 172 (mobile=100vh)

---

## Integration Points

**External Dependencies:**
- Google Fonts API: Fira Code (preconnect in `<head>`)
- No JavaScript libraries/frameworks
- No backend - pure static site

**Links to External Content:**
- GitHub repos, LinkedIn, email in `contact` command
- Certificate links (currently placeholder `#`)
- Live product link: `https://ai5.vn` in `exp` command

---

## Gotchas & Edge Cases

1. **Mobile Keyboard:** Must use `#hidden-input` approach. Regular input breaks UX.
2. **Clear Command:** Special case handled before history (script.js line 124)
3. **Scroll Performance:** `setTimeout` is mandatory, not optional (lines 135, 138, 153)
4. **WhiteSpace:** `#output` uses `pre-wrap` to preserve formatting while allowing wrapping
5. **History Cycling:** ArrowUp/Down only works after first command (historyIndex=-1 initial state)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dinhductai)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/dinhductai)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
