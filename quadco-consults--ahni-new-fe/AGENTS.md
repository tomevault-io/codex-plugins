# Project Cheat Sheet

## 1. Architecture Overview

- **src/**: Main source code directory
  - **components/**: Reusable React components
  - **pages/**: Top-level route components
  - **hooks/**: Custom React hooks
  - **utils/**: Utility/helper functions
  - **services/**: API calls and backend communication
  - **assets/**: Static files (images, styles)
  - **App.js / App.tsx**: Main app entry point
  - **index.js / index.tsx**: ReactDOM render entry

## 2. Key Components & Patterns

- **Functional Components**: Most UI built with function components
- **Hooks**: Usage of `useState`, `useEffect`, and custom hooks
- **Context API**: For global state management (if present)
- **Component Composition**: Components composed for reusability
- **Container/Presentational Pattern**: Separation of logic/UI (if present)

## 3. APIs & Data Flow

- **API Calls**: Centralized in `services/` or `api/` directory
- **Fetch/Axios**: Used for HTTP requests
- **Data Flow**: Data fetched in hooks or useEffect, passed via props/context
- **Error Handling**: Centralized or per-request (check `services/`)

## 4. Reusable Utilities & Helpers

- **utils/**: Contains helper functions (e.g., formatting, validation)
- **hooks/**: Custom hooks for shared logic
- **Common Components**: Buttons, Modals, Inputs, etc. in `components/`

## 5. Frequently Used Components

- Components like `Button`, `Modal`, `Input`, `Form`, and `Loader` are used repeatedly across the project for consistency and reusability.

---

**Tip:** Check `components/` and `utils/` before creating new components or helpers to avoid duplication.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Quadco-Consults) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
