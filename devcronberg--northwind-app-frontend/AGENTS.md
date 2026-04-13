# GitHub Copilot Instructions for Northwind Frontend

## Project Overview
This is a vanilla JavaScript frontend application for Northwind Traders, built with Web Components, HTML5, CSS3, and modern ES6+ JavaScript. The application is a Progressive Web App (PWA) that can be installed on devices.

## Technology Stack
- **Vanilla JavaScript** - ES6+ modules, no frameworks
- **Web Components** - Custom Elements API for reusable components
- **Fomantic UI** - CSS framework for styling (loaded via CDN)
- **jQuery** - Required by Fomantic UI (loaded via CDN)
- **PWA** - Progressive Web App with manifest and service worker

## Code Quality Requirements

### 🚨 CRITICAL: Always Run Lint Before Completion
**Before marking any task as complete or delivering code changes, you MUST:**
1. Run `npm run lint` to check all code
2. Fix any ESLint, HTMLHint, or Stylelint errors
3. Verify all linting passes with no errors
4. Only then consider the implementation complete

**Available lint commands:**
```bash
npm run lint          # Run all linters
npm run lint:html     # Check HTML files
npm run lint:css      # Check CSS files
npm run lint:js       # Check JavaScript files
```

**Never deliver code that fails linting!**

### Code Style Guidelines

#### JavaScript
- Use ES6+ modules with `import`/`export`
- Use `async`/`await` for asynchronous operations
- Use class-based Web Components extending `HTMLElement`
- Follow ESLint configuration in `eslint.config.mjs`
- No unused variables (ESLint will catch this)
- Use `const` and `let`, never `var`
- Use template literals for string concatenation
- Always handle errors in try/catch blocks

#### HTML
- Use semantic HTML5 elements
- Include ARIA attributes for accessibility
- Follow HTMLHint rules in `.htmlhintrc`
- Proper indentation (2 spaces)
- Always include alt text for images

#### CSS
- Follow Stylelint rules
- Use Fomantic UI classes when possible
- Custom CSS only in `css/styles.css`
- Use CSS custom properties for theme colors

## PWA Architecture

### Service Worker (`sw.js`)
- **Minimal implementation** - No caching or offline functionality
- Only provides install/activate events for PWA installability
- Pass-through fetch handler (no cache logic)
- Keep it simple - this is intentional design

### Web App Manifest (`manifest.json`)
- App name: "Northwind Traders"
- Short name: "Northwind"
- Theme color: `#2185d0` (Fomantic UI blue)
- Display mode: `standalone`
- Icons: 192x192 and 512x512 PNG format

### PWA Files
- `manifest.json` - Web app manifest
- `sw.js` - Service worker
- `assets/icon-192.png` - App icon 192x192
- `assets/icon-512.png` - App icon 512x512
- `assets/apple-touch-icon.png` - iOS icon 180x180
- `assets/favicon.ico` - Favicon

### PWA Meta Tags
All HTML files must include:
```html
<meta name="theme-color" content="#2185d0">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="default">
<link rel="manifest" href="manifest.json">
<link rel="icon" type="image/x-icon" href="assets/favicon.ico">
<link rel="apple-touch-icon" href="assets/apple-touch-icon.png">
```

## Component Structure

### Web Components Pattern
All components should follow this structure:
```javascript
class ComponentName extends HTMLElement {
    constructor() {
        super();
        // Initialize state
    }

    connectedCallback() {
        this.render();
        // Fetch data if needed
    }

    render() {
        this.innerHTML = `...`;
        // Attach event listeners after render
    }
}

customElements.define('component-name', ComponentName);
```

### Existing Components
- `app-header` - Navigation header
- `app-footer` - Footer with copyright
- `customer-table` - Customer CRUD operations
- `customer-revenue-table` - Dashboard revenue table
- `form-text-input` - Reusable form input

### Component Guidelines
- Use kebab-case for custom element names (must include hyphen)
- Define components at the bottom of the file
- Import dependencies at the top
- Always clean up event listeners if needed
- Use `this.innerHTML` for rendering (simple approach for this project)

## API Integration

### API Configuration
Located in `js/config/settings.js`:
```javascript
export const API_CONFIG = {
    BASE_URL: 'https://northwind-backend-b088.onrender.com/api',
    TIMEOUT: 30000
};
```

### Backend Behavior
⚠️ **Important:** The backend API is hosted on Render.com free tier and sleeps after 15 minutes of inactivity. First request can take 30-50 seconds.

### API Endpoints
- `GET /api/public/customers` - Fetch all customers
- `GET /api/public/customers-with-revenue` - Fetch customers with revenue
- `GET /api/public/customers/{id}` - Fetch specific customer
- `POST /api/public/customers` - Create new customer
- `PUT /api/public/customers/{id}` - Update customer
- `DELETE /api/public/customers/{id}` - Delete customer

### API Call Pattern
```javascript
try {
    const response = await fetch(`${API_CONFIG.BASE_URL}/endpoint`, {
        method: 'GET',
        headers: { 'Content-Type': 'application/json' },
        signal: AbortSignal.timeout(API_CONFIG.TIMEOUT)
    });
    
    if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const data = await response.json();
    // Handle data
} catch (error) {
    console.error('Error:', error);
    this.error = error.message;
    this.render();
}
```

## File Structure

```text
Northwind.App.Frontend/
├── index.html                    # Dashboard page
├── customers.html                # Customer management page
├── about.html                    # About page
├── manifest.json                 # PWA manifest
├── sw.js                         # Service worker
├── package.json                  # Dev dependencies
├── eslint.config.mjs             # ESLint configuration
├── assets/
│   ├── favicon.ico               # Favicon
│   ├── favicon.svg               # SVG favicon
│   ├── icon-192.png              # PWA icon 192x192
│   ├── icon-512.png              # PWA icon 512x512
│   └── apple-touch-icon.png      # iOS icon 180x180
├── css/
│   └── styles.css                # Custom styles
├── js/
│   ├── app.js                    # Application entry point
│   ├── config/
│   │   └── settings.js           # API configuration
│   └── components/
│       ├── app-header.js
│       ├── app-footer.js
│       ├── customer-table.js
│       ├── customer-revenue-table.js
│       └── form-text-input.js
└── .github/
    └── copilot-instructions.md   # This file
```

## Development Workflow

### When Adding New Features
1. Understand the existing component pattern
2. Follow Web Component structure
3. Use Fomantic UI classes for styling
4. Add error handling for API calls
5. Include loading states
6. Add ARIA labels for accessibility
7. **Run `npm run lint` and fix all errors**
8. Test in browser

### When Modifying Existing Code
1. Read the current implementation first
2. Maintain consistent style with existing code
3. Don't break existing functionality
4. Update event listeners if DOM structure changes
5. **Run `npm run lint` and fix all errors**
6. Test thoroughly

### When Adding New Components
1. Create new file in `js/components/`
2. Follow the component pattern (see above)
3. Import in the relevant HTML file
4. Use `<script type="module">` for imports
5. Define custom element with kebab-case name
6. **Run `npm run lint` and fix all errors**

## Important Patterns

### Modal Pattern (Fomantic UI)
```javascript
const modal = document.createElement('div');
modal.id = 'modal-id';
modal.className = 'ui modal';
document.body.appendChild(modal);

modal.innerHTML = `...`;

$(modal).modal({
    closable: false,
    autofocus: false,
    onApprove: () => {
        // Handle approval
        return false; // Prevent auto-close
    }
}).modal('show');
```

### CRUD Operations Pattern
- **Create:** POST to `/api/public/customers` (no ID in body)
- **Read:** GET from `/api/public/customers` or with `/{id}`
- **Update:** PUT to `/api/public/customers/{id}` (ID in URL and body)
- **Delete:** DELETE to `/api/public/customers/{id}` with confirmation

## Deployment

This project is deployed to GitHub Pages via GitHub Actions workflow (`.github/workflows/deploy.yml`).

**Deployment triggers:**
- Automatic on push to `main` branch
- Runs linting before deployment
- Deployment fails if linting fails

**Live URLs:**
- Frontend: https://devcronberg.github.io/Northwind.App.Frontend
- Backend API: https://northwind-backend-b088.onrender.com

## Common Gotchas

### 1. jQuery and Fomantic UI
- jQuery is required for Fomantic UI modals
- Must load jQuery before Fomantic UI
- Use `$()` for Fomantic UI components
- Access via CDN (not npm packages)

### 2. Custom Elements
- Custom element names MUST contain a hyphen
- Use `connectedCallback()` not `mounted()`
- Define elements only once (check if already defined)

### 3. Service Worker
- Keep it minimal (no caching in this project)
- Don't add caching logic unless explicitly requested
- Always test with `npm run lint:js`

### 4. Modal ID Handling
- `customerId` from data attributes is string
- Convert to number with `parseInt(customerId, 10)`
- API returns numbers, DOM attributes are strings

### 5. Form Data Collection
- Query all `form-text-input` elements
- Use `.getAttribute('name')` for field names
- Use `.value` property for values
- Skip `customerId` field when creating new records

## Testing Checklist

Before completing any task:

- Code follows existing patterns
- No console errors in browser
- API calls work correctly
- Loading states display properly
- Error messages display to user
- Responsive design maintained
- ARIA labels present
- **All linting passes (`npm run lint`)**
- No unused variables or parameters

## Questions to Ask Before Implementation

1. Does this change affect multiple components?
2. Do I need to update event listeners?
3. Should I add error handling?
4. Does this need a loading state?
5. Is this accessible (ARIA labels)?
6. Does this follow the existing pattern?
7. **Will this pass linting?**

## Remember

- **ALWAYS run `npm run lint` before marking tasks complete**
- Keep it simple - this is a demo/learning project
- No caching or offline logic in service worker
- Follow existing patterns - consistency is key
- jQuery is only for Fomantic UI modals
- All API calls need error handling
- Component names must have hyphens
- Test in browser after changes

---

*Updated: January 24, 2026*
*Version: 2.1 (Refined README and instructions)*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/devcronberg)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/devcronberg)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
