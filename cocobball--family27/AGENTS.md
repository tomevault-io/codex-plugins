# Copilot Instructions — Family Home Dashboard

These instructions are authoritative.
GitHub Copilot MUST follow them exactly when generating, editing, or refactoring code in this repository.

If there is any ambiguity, Copilot must prefer architecture safety and correctness over features or convenience.

---

## 0. Absolute Rules (Non-Negotiable)

1. The dashboard must NEVER boot into a blank UI.
2. ONE unified localStorage key only: family_dashboard_db_v1.
3. Modules are code. Windows are layout.
4. App.jsx is a window manager ONLY.
5. Drop-in modules must work without editing App.jsx or registries.

---

## 1. Core Mental Model

This project is a dashboard OS, not a traditional SPA.

---

## 2. Tech Stack (Do Not Change)

- React + Vite
- Tailwind CSS
- lucide-react icons
- LocalStorage ONLY
- No Redux

---

## 3. Storage Rules

All storage uses the unified key family_dashboard_db_v1.
Storage logic lives in src/core/dashboardStore.js.

Modules must not access localStorage directly.

---

## 4. App.jsx Rules

App.jsx handles window management only.
No module logic or schemas.

---

## 5. Module Contract

Modules export moduleDef with id, title, Component.

---

## 6. Module Structure

src/modules/<moduleId>/
index.js
module.jsx
helpers.js

---

## 7. Boot Safety

The dashboard must never be empty.
Starter modules must be enabled automatically.

---

## FINAL RULE

Modules are code.
Windows are layout.
The dashboard must never be empty.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cocobball)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/cocobball)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
