# Workspace Custom Instructions

These instructions help Copilot tailor code recommendations and answers specifically for the Task Manager workspace.

## Technology Stack & Guidelines

*   **Architecture**: Vanilla HTML5, CSS3, and JavaScript (ES6+ Modules).
*   **No Frameworks**: Do not introduce React, Vue, Angular, or jQuery. Stick to vanilla DOM APIs.
*   **Vanilla CSS**: Use CSS Custom Properties (variables) for the design system. Avoid utility-first CSS frameworks like Tailwind.
*   **CSS Standards**: Design-first approach using flexbox/grid layout, smooth CSS animations (`transition`), and glassmorphism styling.
*   **Modularity**: JavaScript files should be modular and structured by concerns:
    *   `js/storage.js` for data persistence
    *   `js/notifications.js` for toast feedback
    *   `js/app.js` for DOM management, main event listeners, and UI logic

## Coding Standards

*   Use semantic HTML elements (`<header>`, `<main>`, `<section>`, `<article>`, `<dialog>`).
*   Always use ES6+ features (e.g. `const`/`let`, arrow functions, destructuring, modules).
*   Ensure functions are well-commented and focus on a single responsibility.
*   Prefer descriptive naming conventions (e.g. `updateTaskCounter()` instead of `count()`).
*   All interactive elements must have unique, descriptive `id` or `class` attributes.

## Design Patterns

*   **Theme**: Dark-first modern UI with high-contrast accent colors (indigo, violet, emerald).
*   **Feedback**: Any write/update action (add, edit, delete, complete) should trigger a toast notification.
*   **Aesthetics**: Glassmorphism cards (`background: rgba(...)`, `backdrop-filter: blur()`).
*   **Micro-interactions**: Hover effects, smooth checkbox scales, list entry transitions.

---
> Source: [Dhanalakshmi-1604/Task-Manager-App](https://github.com/Dhanalakshmi-1604/Task-Manager-App) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-08 -->
