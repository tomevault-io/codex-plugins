# Copilot instructions for ActivitesClasse

## General project rules

These rules apply across the whole project:

- Prefer reusing shared hooks, utilities, and components over duplicating similar logic with small variations.
- Generated code must stay **clear, readable, and easy to debug**, with meaningful naming. In the frontend, add explicit `id` attributes on key UI blocks and controls (`div`, `section`, `input`, `button`, etc.) whenever useful for debugging and inspection.
- All user-facing text (labels, buttons, instructions, placeholders, success/error messages, and activity content) must be written in **French**, with correct spelling, punctuation, and accents (`é`, `è`, `à`, `ç`, etc.). Avoid mixed French/English wording in the UI unless technically required.
- Before declaring the work complete, verify relevant changes with `docker compose up --build -d`.

## Frontend management panels

When modifying admin or teacher management panels in `frontend/src/components/*ManagementPanel.js` or related views in `frontend/src/views/`:

- Keep **strict behavioral parity** across analogous panels: `StudentsManagementPanel`, `GroupManagementPanel`, `TeachersManagementPanel`, and `ClassesManagementPanel`.
- If one panel is used as the reference, mirror **both UI and behavior** exactly: selection logic, delete actions, disabled states, feedback messages, and fade/auto-dismiss transitions.
- Success messages must use the shared `useAutoDismissMessage` hook and Tailwind transition classes: `transition-opacity duration-500`, `opacity-0`, `opacity-100`.
- For list management panels, preserve the established interaction pattern: `Supprimer Tout` in the header and `Supprimer` only for the currently selected item.

## React and Tailwind conventions

When modifying React components in `frontend/src/**`:

- Prefer functional components with clear hooks usage and avoid unnecessary state duplication.
- Keep components focused and readable; extract repeated UI or logic into shared hooks, helpers, or subcomponents.
- Use controlled inputs for forms and keep handler names explicit (`handleAddStudent`, `handleDeleteClass`, etc.).
- Preserve existing naming patterns, prop naming, and component structure unless a broader refactor is explicitly needed.
- Add meaningful `id` attributes on important containers and controls to simplify inspection, debugging, and future UI testing.
- Keep Tailwind class usage consistent with the surrounding codebase and favor readable utility groupings over dense inline styling changes.
- For feedback messages and action buttons, preserve the established visual language and disabled/loading states across similar screens.

## Interactive activities conventions

When modifying activities in `frontend/src/activities/**`:

- Keep a **consistent structure and rendering** across activities whenever possible, reusing the same major sections and order: title/context, instructions, main interaction area, feedback/result area, and action buttons.
- Prefer existing activity layout patterns before introducing one-off UI structures; if a different layout is necessary, keep the same visual language and overall navigation flow.
- All visible activity text must be in **French** and preserve proper accents and child-friendly wording. Use consistent labels such as `Consigne`, `Valider`, `Réessayer`, `Continuer`, `Bravo !`, etc. when relevant.
- Preserve clear loading, empty, and error states in French so the activity remains understandable in classroom conditions.
- Keep controls simple, readable, and easy to use on classroom devices, with coherent button placement and explicit feedback after user actions.
- When an activity exposes default configurable content or instructions, keep those defaults in French and align them with the tone and pedagogical style of the existing activities.
- Don’t overload it with irrelevant information; it should remain simple and uncluttered.

## Backend routes and data integrity

When modifying the backend in `backend/server.js`, `backend/db.js`, or `backend/routes/*.js`:

- Keep route handlers explicit, readable, and consistent with the existing Express style.
- Validate request inputs early and return **explicit French error messages** when a request is invalid or cannot be completed.
- When deleting entities with dependencies, clean up related data first and avoid leaving orphaned records.
- Preserve data integrity between teachers, classes, groups, students, activities, and results when adding, updating, or deleting data.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sletonqu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
