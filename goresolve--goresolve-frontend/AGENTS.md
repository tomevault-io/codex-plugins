## Patterns to Follow

### Preferred Patterns ✅

- Prefer semantic HTML elements (`<button>`, `<label>`, `<strong>`, `<em>`, `<p>`, etc.) over generic `<span>` elements.
- Use route **name** instead of path when navigating (e.g., `router.push({ name: 'home' })` instead of `router.push('/home')`).
- Choose heading levels (`<h1>`–`<h6>`) based on document structure and semantics; adjust visual size with Tailwind utility classes, not by picking a different heading tag.
- Use inline arrow functions instead of creating a named function that contains only a single line of logic.
- Using routes opt to identify it with the route name rather than the path. For example, use `router.push({ name: 'home' })` instead of `router.push('/home')`. This allows for better maintainability and readability of the code.
- Modals are always the first thing in the template.
- Elements that aren't always visible or shouldn't be rendered until necessary (e.g., modals, dropdowns, tooltips) should be organized all together wrapped by a ref under the name of "hiddenContent".
- Temporary form values should be organized together under a ref named "tempValues".
- Data that defines state types (e.g., `isLoading`, `isOpen`, `isEditing`) should be organized together under a ref named "metadata".
- Create a secondary variable inside stores to hold displayed items of entities from backend (e.g., `displayedUsers`, `displayedProjects`) to avoid mutating the original data and to allow for easier filtering and sorting of displayed items without affecting the source data.
- Functions are wrote as arrow functions and organized in the following order: 1. functions that are called by the template, 2. functions that are called by other functions, 3. lifecycle hooks.
- Edit states should be triggered by doubleclicking on the element that is being edited. This is a common and intuitive user interaction pattern for entering edit mode.

### Patterns to Avoid ❌

- Using `<span>` as a generic wrapper when a more appropriate semantic element exists.
- Changing heading levels just to change visual size.
- Creating functions that contain only a single line of logic; prefer using an inline arrow function instead.
- Inside `<template>` tags, avoid using .value when accessing refs (e.g., use `metaData.isEditing` instead of `metaData.value.isEditing`).
- Never use `<span>` to write text that should be wrapped in a more semantic element (e.g., `<p>`, `<strong>`, `<em>`) but never `<div>` or `<button>`.
- No need to import Vue components in the `<script setup>` tag, as they are automatically imported by Vue.
- Using buttons to activate edit mode; edit states should be triggered by double-clicking the element being edited.
- Never use single letter variable names anywhere — use descriptive names that indicate the purpose of the variable ALWAYS.

## Code Style & Conventions

## Error Handling

- Errors are handled right away as they're called (e.g., stores, `.ts` files) and not passed up to the caller. This is because we want to avoid multi-levels of error handling.

## Comments & Documentation

- Comment blocks should be in the beginning of the function scope and used to explain "why" something is done, not "what" is being done. The code itself should be clear enough to explain the "what".

### Naming Conventions

- Components: `UserCard.vue`, `AuthForm.vue`
- Hooks: `useAuth.ts`, `useDebounce.ts`
- Utils: `formatDate.ts`, `validateEmail.ts`
- Tests: `*.test.ts` co-located with source files
- Interfaces: Should start with an I (e.g. `interface IUser { ... }`)

### Vue script tag order

- The order of the script tags are:
    1. refs and computed properties.
    2. methods.
    3. watchers and other reactive properties that require triggers (e.g. click outside events).
    4. lifecycle hooks.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GoResolve)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/GoResolve)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
