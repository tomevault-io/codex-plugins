# Cursor Instructions – Expo Project

These instructions guide Cursor (AI pair programmer) when working in this repository. Follow them strictly.

## Project Overview

* **Framework**: Expo (React Native)
* **Language**: TypeScript
* **Platform targets**: iOS, Android, Web (via Expo)
* **Styling**: Use existing styling approach in the project (e.g. StyleSheet, NativeWind, or other). Do not introduce new styling libraries unless explicitly requested.

## General Rules

* Prefer **clarity, simplicity, and maintainability** over clever abstractions.
* Make **minimal, targeted changes** that directly address the request.
* Do **not** refactor unrelated code.
* Avoid speculative features or over-engineering.
* Match the **existing code style and conventions**.

## TypeScript & React Native Guidelines

* Use **functional components** only.
* Always type props, state, and function return values when non-trivial.
* Avoid `any`. If unavoidable, explain why.
* Prefer `useCallback` and `useMemo` **only when justified**.
* Keep components small and focused.

## Expo-Specific Guidelines

* Use **Expo APIs** instead of custom native implementations whenever possible.
* Do not add native code or custom dev clients unless explicitly requested.
* Respect Expo-managed workflow constraints.
* Prefer `expo-router` conventions if the project uses it.

## File & Folder Conventions

* Do not create new folders unless necessary.
* Reuse existing utilities, hooks, and components before creating new ones.
* Name files consistently with existing patterns.

## Dependencies

* Do not add new dependencies without a clear reason.
* If a dependency is required, explain:

  * Why it is needed
  * Why existing dependencies are insufficient

## Performance & UX

* Avoid unnecessary re-renders.
* Handle loading, error, and empty states where relevant.
* Ensure UI changes are responsive and mobile-friendly.

## Error Handling

* Fail gracefully.
* Prefer user-friendly error messages.
* Log errors only when useful for debugging.

## Testing & Safety

* Do not break existing functionality.
* If a change may introduce risk, call it out explicitly.
* Avoid destructive operations unless explicitly requested.

## Communication Style

* When responding:

  * Explain *what* was changed and *why*.
  * Be concise and technical.
  * Do not repeat the user’s request verbatim.

## Code Cleanliness & Consistency (Strict)

* Do **not** add:

  * Extra comments a human developer would not reasonably add
  * Comments that are inconsistent with surrounding files
* Do **not** introduce:

  * Defensive checks that are unnecessary for trusted or validated code paths
  * `try/catch` blocks that are abnormal for that area of the codebase
* **Never** cast to `any` to work around type issues
* Avoid workaround patterns that hide type or logic problems instead of fixing them
* Match existing:

  * Error-handling philosophy
  * Null/undefined assumptions
  * Control flow style
* If an inconsistency is unavoidable, explicitly call it out before making the change

## Out of Scope

* Do not:

  * Rewrite the project architecture
  * Migrate frameworks or languages
  * Add analytics, auth, or payments unless asked

---

## UI / UX Style Guide

### Design Principles

* Visual style is **clean, minimal, and modern**
* Color palette is **white and soft black (near-black)** — never pure black
* UI must work seamlessly in **both light and dark mode**
* Prioritize **clarity, hierarchy, and usability** over decoration
* Always think from the **end-user’s perspective**

### Color System

* **Light mode**:

  * Background: pure white or off-white
  * Text: near-black (e.g. zinc-900)
  * Borders & dividers: subtle gray (zinc-200 / zinc-300)
* **Dark mode**:

  * Background: soft near-black (e.g. zinc-950 / zinc-900)
  * **Never use pure black for backgrounds**
  * Text: off-white (zinc-50 / zinc-100)
  * Borders & dividers: zinc-800
* Avoid strong contrast extremes in dark mode
* Accent colors should be subtle and used sparingly

### Typography

* Use system fonts (default React Native fonts)
* Clear hierarchy:

  * Headlines: larger, semibold
  * Body text: regular weight, readable size
  * Secondary text: muted color, smaller size
* Line height must remain comfortable for mobile reading

### Layout & Spacing

* Use **consistent spacing scale** via NativeWind utilities
* Prefer padding and margin over absolute positioning
* Align content clearly; avoid visual clutter
* Use vertical rhythm generously

### Components

* **Always prefer components** when it makes sense:

  * Buttons
  * Inputs
  * Cards
  * Lists / list items
  * Headers / section titles
  * Empty states
* Components must:

  * Be reusable
  * Have a single clear responsibility
  * Follow consistent API and naming

### Styling Rules (Strict)

* **ALWAYS use NativeWind classes for all styling**
* Do not mix StyleSheet or inline styles
* Do not introduce alternative styling systems
* Use `className` exclusively

### Buttons

* Clear visual hierarchy (primary / secondary / destructive)
* Adequate touch targets (minimum 44px height)
* Disabled states must be visually distinct

### Forms & Inputs

* Labels must be clear and readable
* Inputs must have visible focus states
* Error states should be helpful and non-intrusive

### Icons

* **Use lucide-react-native icons only**
* Icons should support light and dark mode
* Icon usage must be purposeful, not decorative

### Accessibility & UX

* Touch targets must be finger-friendly
* Text must remain readable in all themes
* Avoid hidden gestures without affordances
* Empty, loading, and error states must be designed

### Consistency Rules

* UI patterns must be reused, not reinvented
* Do not introduce new visual styles without updating this guide
* If a screen deviates intentionally, call it out explicitly

---

**All UI changes must comply with this style guide. If a request conflicts with it, flag the conflict before proceeding.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Quintenntje) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-17 -->
