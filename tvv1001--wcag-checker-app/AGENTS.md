# GitHub Copilot Instructions for WCAG Checker App

## Project Overview

This is a web application that checks URLs for WCAG (Web Content Accessibility Guidelines) compliance using axe-core. The app supports scanning web pages for accessibility issues from WCAG 2.0 A through WCAG 2.2 AA standards.

## Tech Stack

- **Backend**: Node.js with Express
- **Browser Automation**: Puppeteer and Chrome Launcher
- **Accessibility Testing**: axe-core
- **Frontend**: Vanilla JavaScript (ES6+), HTML5, CSS3
- **Module System**: ES6 modules for frontend code

## Code Style and Conventions

### JavaScript

- Use modern ES6+ syntax (arrow functions, async/await, const/let, template literals)
- Prefer `async/await` over promise chains for asynchronous operations
- Use `const` by default; only use `let` when reassignment is necessary
- Use template literals for string interpolation
- Add JSDoc comments for exported functions and complex logic
- Use camelCase for variable and function names
- Use descriptive variable names that clearly indicate purpose

### File Organization

```
src/
  ├── index.html          # Main HTML entry point
  ├── scripts/            # JavaScript modules
  │   ├── app.js         # UI interaction and event handlers
  │   └── checker.js     # Accessibility checking logic
  ├── styles/            # CSS stylesheets
  │   └── main.css       # Main styles
  └── utils/             # Utility functions and configurations
      └── axe-config.js  # axe-core configuration presets
server.js               # Express server with /api endpoints
```

### Backend Patterns

#### API Endpoints

- Use RESTful conventions
- Prefix API routes with `/api/`
- Return JSON responses with appropriate HTTP status codes
- Include error handling with descriptive error messages
- Log important operations to console with contextual prefixes (e.g., `[Scan Request]`)

#### Error Handling

- Validate input parameters early and return 400 for bad requests
- Use try-catch blocks for async operations
- Always clean up resources (browsers, pages) in finally blocks
- Log errors with context before returning error responses

#### Browser Automation

- Use headless Chrome with appropriate flags: `--headless=new`, `--disable-gpu`, `--no-sandbox`
- Connect Puppeteer to launched Chrome instance using debugging port
- Always close pages and browsers in finally blocks to prevent resource leaks
- Set reasonable timeouts for page navigation and interactions
- Inject axe-core and configuration before running tests

### Frontend Patterns

#### DOM Manipulation

- Cache DOM element references in variables at module scope
- Use `document.getElementById()` or `document.querySelector()` for element selection
- Use `addEventListener()` for event binding
- Clear and rebuild containers when displaying new results

#### URL Handling

- Use URLSearchParams for query parameter manipulation
- Update browser history with `pushState` when performing scans
- Encode URLs properly when adding to query parameters
- Load and auto-run scans if URL parameter is present on page load

#### Dynamic Imports

- Use dynamic imports `await import()` for code splitting when appropriate
- Import checker functionality dynamically to reduce initial load

### CSS

- Use semantic class names (e.g., `.issue`, `.results-container`)
- Follow mobile-first responsive design principles
- Ensure sufficient color contrast for accessibility
- Use modern CSS features (flexbox, grid) for layouts

## Accessibility Best Practices

Since this app checks for accessibility compliance, it must itself be accessible:

- Use semantic HTML elements (`<button>`, `<main>`, `<section>`, etc.)
- Include proper ARIA labels where necessary
- Ensure all interactive elements are keyboard accessible
- Maintain sufficient color contrast (WCAG AA minimum)
- Provide meaningful alt text for images
- Use proper heading hierarchy (h1 → h2 → h3)
- Ensure form inputs have associated labels
- Make sure error messages are clear and accessible

## Security Considerations

- Validate and sanitize URL inputs
- Only allow `http:` and `https:` protocols
- Use Content Security Policy where appropriate
- Never execute arbitrary code from user input
- Set appropriate CORS headers if needed
- Use environment variables for sensitive configuration (e.g., `process.env.PORT`)

## Testing and Validation

- Test with various URL formats (with/without protocols, international characters)
- Verify proper resource cleanup to prevent memory leaks
- Check error handling for edge cases (unreachable URLs, timeouts, malformed responses)
- Ensure UI remains responsive during long-running scans
- Test keyboard navigation and screen reader compatibility

## Common Patterns in This Project

### axe-core Configuration

- Use preset configurations from `axe-config.js` module
- Support multiple WCAG levels: wcag2a, wcag2aa, wcag21a, wcag21aa, wcag22aa
- Return violations with rule ID, description, impact, and affected elements

### Server-Side Scanning Flow

1. Validate URL parameter
2. Launch headless Chrome
3. Connect Puppeteer to Chrome instance
4. Navigate to target URL
5. Inject axe-core library
6. Inject axe configuration
7. Run accessibility tests
8. Format and return results
9. Clean up browser resources

### Client-Side Display Flow

1. Capture URL from input field
2. Call `/api/scan` endpoint with URL parameter
3. Display loading state or progress indicator
4. Parse and display results in user-friendly format
5. Update browser URL with query parameter
6. Handle and display errors gracefully

## Dependencies

- **axe-core**: Core accessibility testing library
- **express**: Web server framework
- **puppeteer**: Browser automation
- **chrome-launcher**: Chrome instance management

## Environment and Configuration

- Default port: 8080 (configurable via `PORT` environment variable)
- Static files served from `src/` directory
- API routes under `/api/` prefix

## When Adding New Features

- Keep the UI simple and focused on accessibility scanning
- Add new axe-core configurations to `axe-config.js`
- Follow existing error handling and logging patterns
- Update README.md if user-facing features are added
- Ensure new features maintain or improve accessibility
- Use descriptive commit messages

## Code Examples

### Adding a New API Endpoint

```javascript
app.get("/api/endpoint", async (req, res) => {
  try {
    // Validate input
    if (!req.query.param) {
      return res.status(400).json({ error: "Missing parameter" });
    }

    // Process request
    const result = await processRequest(req.query.param);

    // Return response
    res.json({ data: result });
  } catch (error) {
    console.error(`[Endpoint] Error: ${error.message}`);
    res.status(500).json({ error: "Internal server error" });
  }
});
```

### Display Function Pattern

```javascript
function displayResults(results) {
  resultsContainer.innerHTML = "";
  if (results.length === 0) {
    resultsContainer.innerHTML = "<p>No issues found.</p>";
  } else {
    results.forEach((item) => {
      const element = document.createElement("div");
      element.classList.add("item-class");
      element.innerHTML = `<strong>${item.title}</strong>: ${item.description}`;
      resultsContainer.appendChild(element);
    });
  }
}
```

## Git Workflow

- Keep commits focused and atomic
- Write clear commit messages
- Test changes before committing
- Follow gitignore patterns (node_modules, .env, logs)

---
> Source: [tvv1001/wcag-checker-app](https://github.com/tvv1001/wcag-checker-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-27 -->
