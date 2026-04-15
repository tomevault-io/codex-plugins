# Copilot Instructions for set08801

## Project Overview
This is a web-based coursework project using vanilla HTML, CSS, and JavaScript. The codebase is structured as a single-page application with minimal dependencies.

## Architecture

### File Organization
- **index.html** - Single entry point; contains DOM structure and semantic HTML
- **style.css** - All styling; separate from HTML (do not use inline styles)
- **script.js** - All client-side logic and DOM manipulation
- **README.md** - Project documentation and coursework requirements

### Code Structure
- Keep HTML clean and semantic; avoid presentation logic in markup
- Use CSS classes for styling; keep specificity low and use BEM-like naming for complex components
- Implement all interactivity in `script.js` using vanilla JavaScript (no frameworks)

## Development Conventions

### JavaScript Patterns
- Use `const`/`let` (never `var`)
- Prefer event delegation on document or parent containers for dynamic elements
- Cache DOM queries to avoid repeated selections:
  ```javascript
  const button = document.getElementById('submit');
  button.addEventListener('click', handleSubmit);
  ```
- Use clear, descriptive function names that describe the action (`handleClick`, `updateUI`, `validateForm`)

### CSS Patterns
- Use a flat structure; avoid deep nesting
- Define custom properties for repeated values (colors, spacing)
- Mobile-first approach: start with mobile styles, add media queries for larger screens
- No absolute positioning unless specifically required

### HTML Patterns
- Use semantic HTML5 elements (`<button>`, `<form>`, `<section>`, etc.)
- Include `aria-` attributes for accessibility
- Keep markup lean; use CSS for layout, JavaScript for interaction

## Workflows

### Local Development
- Open `index.html` directly in a browser or use a simple HTTP server:
  ```powershell
  python -m http.server 8000
  # or
  python3 -m http.server 8000
  ```
- Test in browser developer console for JavaScript errors
- Use browser DevTools to debug CSS and inspect DOM

### Testing
- Manually test all user interactions in multiple browsers (Chrome, Firefox, Safari, Edge)
- Check responsive design at breakpoints: 320px, 768px, 1024px
- Verify accessibility with keyboard navigation and screen reader testing

## Common Tasks

### Adding a Feature
1. Add HTML structure to `index.html` (semantic elements only)
2. Style in `style.css` (mobile-first)
3. Add event listeners and logic in `script.js`
4. Test cross-browser compatibility

### Debugging
- Use browser DevTools to inspect elements and debug JavaScript
- Check browser console for errors and warnings
- Use `console.log()` for variable inspection (remove before commits)

## Documentation
- Update `README.md` with new features, setup instructions, or coursework requirements
- Comment complex logic in `script.js` but keep comments minimal and focused on "why" not "what"
- No comments needed for obvious code

## Quality Standards
- Validate HTML with W3C validator
- Ensure no console errors or warnings
- Test on actual devices or emulators if possible
- Follow accessibility standards (WCAG 2.1 Level AA)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Obioma-Nwabuisi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
