# Copilot Instructions for Schedule Generator

## Project Overview
This is a single-page web application for generating university schedules based on user-defined subjects, groups, and priorities. The app is built with vanilla JavaScript, HTML, and CSS. All state is managed in-browser using `localStorage`.

## Architecture & Data Flow
- **Entry Point:** `index.html` loads `app.js` and `styles.css`.
- **State Management:**
  - All user data (subjects, groups, priorities) is stored in `localStorage` under the key `scheduleGenData`.
  - The main state object is loaded, mutated, and saved via helper functions in `app.js`.
- **UI Components:**
  - Tabs for navigation: Subjects, Priorities, Schedule Generator.
  - Modal dialogs for editing/adding subjects and groups.
  - Drag-and-drop lists for setting priorities.
  - Schedule tables are dynamically generated based on current state and priorities.
- **Theme Support:**
  - Light/dark mode toggled via a button, persisted in `localStorage`.

## Developer Workflows
- **No build step required.**
- **No external dependencies.**
- **Debugging:**
  - Use browser DevTools for inspecting state, DOM, and localStorage.
  - To reset app state, use the "Eliminar todos los datos" button or clear `localStorage` manually.
- **Testing:**
  - Use the "Cargar datos de prueba" button to populate the app with sample data for quick testing.

## Project-Specific Patterns
- **State Mutations:**
  - Always call `updateState()` after mutating the `state` object to persist changes and re-render the UI.
- **UI Updates:**
  - All UI rendering is handled by functions in `app.js` (`renderMaterias`, `renderPrioridad`, `generateAllSchedules`, etc.).
  - The main render function is `renderAll()`, called on page load and after any state change.
- **Schedule Generation:**
  - Schedules are generated in two modes: normal and with lunch break, using custom logic in `generateAllSchedules()`.
  - Priority order for subjects and professors affects schedule selection.
- **Modals:**
  - Modal dialogs are used for all CRUD operations on subjects and groups.

## Conventions & Integration Points
- **All logic is in `app.js`; no frameworks or modules.**
- **UI is fully client-side; no backend or API calls.**
- **Spanish language is used for UI labels and prompts.**
- **CSS uses BEM-like class naming for clarity.**

## Key Files
- `index.html`: Main HTML structure and UI entry point.
- `app.js`: All application logic, state management, and rendering.
- `styles.css`: Theming and layout.

## Example Patterns
- To add a new subject:
  - Use the "Agregar materia" button, fill out the modal form, and submit.
- To change priorities:
  - Drag and drop items in the priority lists; changes are saved automatically.
- To generate schedules:
  - Use the buttons in the "Generador de horarios" tab; results are shown in tables below.

---
**For AI agents:**
- Always update state via provided functions, not direct DOM manipulation.
- Use sample data and UI buttons for quick testing and debugging.
- Reference `app.js` for all logic and patterns; avoid introducing frameworks or build steps.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/its-camilo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
