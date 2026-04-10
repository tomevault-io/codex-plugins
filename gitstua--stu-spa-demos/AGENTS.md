## stu-spa-demos: AI Coding Agent Instructions

### Project Overview
This repo is a collection of single-page apps (SPAs) with a modern, visually appealing style. Each HTML file is a standalone SPA demo, with no build step or framework required. The architecture is flat: each SPA is a separate HTML file in the root directory, with shared assets in `img/`, `svg/`, and subfolders for specialized content.

### Key Conventions
- **Navigation:** Every SPA page must include a prominent "back to home" link at the top, pointing to `/`.
- **Data Storage:** Store all app state in the URL fragment (`window.location.hash`). Do not use cookies, localStorage, or server-side storage.
- **Footer:** Add a floating footer to every page: `Made with ❤️ by Stu at github.com/gitstua/stu-spa-demos using GitHub Copilot`. Increment a visible version number in the footer on every new prompt or code change to the specific spa.
- **Page Explanation:** At the bottom of each SPA, include a clear explanation of what the page does and how it works.
- **Index Page:** `/index.html` must list and link to all SPA demos, with a short description for each.
- **Style:** Use the color palette and layout conventions found in `llm.html` and other recent SPAs. Prioritize clarity, accessibility, and a modern look.

### Patterns & Examples
- **Component Structure:** Each SPA is a single HTML file with embedded CSS and JS. See `llm.html` for a model example: semantic HTML, custom styles, and interactive JS.
- **Tokenization Demo:** The `llm.html` file demonstrates how to build interactive playgrounds and mock pipelines. Use similar patterns for new demos.
- **SVG & Images:** Use assets from `svg/` and `img/` for icons and illustrations. Reference them with relative paths.
- **Specialized Data:** For apps needing data (e.g., `dare-ya-cobber/cards.json`, `japanese/words.csv`), load via fetch and keep logic in the SPA file.

### Developer Workflow
- No build tools or package managers required. Edit HTML/CSS/JS directly.
- Debug in-browser; use browser dev tools for inspection.
- To add a new SPA: create a new HTML file in the root, follow conventions above, and add a link/description to `index.html`.
- For shared logic, copy patterns from existing SPAs rather than extracting shared JS files.

### Integration Points
- No external dependencies except for assets in `img/`, `svg/`, and subfolders.
- Audio demos (e.g., `TempleBell-SoundBible.com-756181215.mp3`, `japanese/audio_files/`) should use native HTML5 audio APIs.

### Example: Adding a New SPA
1. Create `new-demo.html` in the root.
2. Add a back-to-home link at the top.
3. Store all state in the URL fragment.
4. Add a floating footer with the required text and version number.
5. Explain the page at the bottom.
6. Link from `index.html` with a short description.

---
If any section is unclear or missing details, please provide feedback so instructions can be improved for future AI agents.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gitstua)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/gitstua)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
