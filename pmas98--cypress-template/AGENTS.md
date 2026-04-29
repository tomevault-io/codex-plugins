
# The Internet Cypress Template - Project Overview

## What This Project Is

This is a **Cypress E2E testing template** targeting [The Internet](https://the-internet.herokuapp.com/), a demo site by Elemental Selenium for practicing automation. The project demonstrates best practices for maintainable, reusable end-to-end tests.

## Architecture: Page Object Model (POM)

**All tests MUST use the Page Object Model.** Do not put selectors or page logic directly in test files.

### Structure

```
cypress/
├── pages/           # Page Objects - one per page/screen
│   ├── LoginPage.js
│   ├── SecurePage.js
│   ├── HomePage.js
│   ├── DropdownPage.js
│   ├── CheckboxesPage.js
│   └── index.js     # Central exports
├── e2e/             # Test specs - use page objects only
│   ├── login.cy.js
│   ├── secure.cy.js
│   ├── home.cy.js
│   ├── dropdown.cy.js
│   └── checkboxes.cy.js
└── support/
    ├── commands.js  # Reusable commands (e.g. cy.login())
    └── e2e.js
```

### Page Object Rules

- **One page object per page/screen** – encapsulate selectors and actions
- **Export singleton instances** – `export default new LoginPage()`
- **Return `this` for chaining** – `return this` in action methods
- **No assertions in page objects** – assertions belong in tests
- **Use getters for elements** – `get loginButton() { return cy.get('button[type="submit"]'); }`

### Test Rules

- **Tests should be reusable** – compose flows from page object methods
- **Use custom commands for common setup** – e.g. `cy.login()` instead of repeating login steps
- **One concern per test** – keep tests focused and readable
- **Import pages from `../pages` or `../pages/index`** – never duplicate selectors

### The Internet - Key Pages & Credentials

- **Login** (`/login`): tomsmith / SuperSecretPassword!
- **Secure Area** (`/secure`): After successful login
- **Dropdown** (`/dropdown`): Option 1, Option 2
- **Checkboxes** (`/checkboxes`): Two checkboxes (1 unchecked, 2 checked by default)

### Adding New Tests or Pages

1. **New page** → Create `cypress/pages/NewPage.js`, add to `index.js`
2. **New reusable flow** → Add custom command in `commands.js`
3. **New test** → Create `cypress/e2e/feature.cy.js`, use existing page objects

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pmas98) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
