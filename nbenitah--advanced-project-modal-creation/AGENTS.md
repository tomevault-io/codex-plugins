# Copilot Instructions for AI Agents

## Project Overview
This is a simple frontend JavaScript project with the following structure:
- `index.html`: Main HTML file, includes navigation, landing, and project sections. References `assets/style.css` for styling and loads Font Awesome from CDN.
- `app.js`: Contains JavaScript logic. Currently, it fetches and logs the first six incomplete TODOs from an external API (`jsonplaceholder.typicode.com`).
- `assets/style.css`: Custom styles for layout, navigation, and project sections. Uses an orange accent color for highlights.

## Key Patterns & Conventions
- **JavaScript**: All logic is in `app.js`. Uses async/await and fetch for API calls. No frameworks or build tools are used.
- **Styling**: All styles are in `assets/style.css`. Uses BEM-like class naming (e.g., `project_description--link`).
- **HTML**: Semantic structure with clear separation of navigation, landing, and project sections. Social links use Font Awesome icons.
- **External Dependencies**: Font Awesome is loaded via CDN in `index.html`. The only API used is `jsonplaceholder.typicode.com` for demo data.

## Developer Workflows
- **No build step**: Open `index.html` directly in a browser to view the site.
- **Debugging**: Use browser DevTools (Console, Network) to debug JavaScript and inspect styles.
- **No tests or automation**: There are no test scripts or build tools configured.

## Project-Specific Guidance
- **API Usage**: See `firstsixincomplete(userId)` in `app.js` for the pattern to fetch and filter data from an external API.
- **Adding Features**: Place new JavaScript in `app.js` and new styles in `assets/style.css`. Reference new scripts in `index.html` if needed.
- **Assets**: Place images and other static files in the `assets/` directory.
- **Class Naming**: Follow the existing BEM-like convention for new CSS classes.

## Example: Fetching Data
```js
async function firstsixincomplete(userId) {
    const response = await fetch("https://jsonplaceholder.typicode.com/todos");
    const result = await response.json();
    const incompletetasks = result.filter(element => !element.completed).slice(0, 6);
    console.log(incompletetasks);
}
```

## Key Files
- [`index.html`](../index.html): Main HTML structure
- [`app.js`](../app.js): JavaScript logic
- [`assets/style.css`](../assets/style.css): Styles

---
If you add new workflows or conventions, update this file to keep AI agents productive.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nbenitah)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/nbenitah)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
