# Copilot Instructions for AI Agents

## Project Overview

- **Atrium Blog Dashboard** is a React + Vite SPA for managing blog posts with secure authentication and role-based access.
- **Backend:** Supabase (PostgreSQL, RLS policies) for data, Clerk for authentication.
- **Frontend:** React (hooks, context), Vite, Styled Components, React Query, React Router DOM.

## Architecture & Data Flow

- **Authentication:**
  - Clerk manages user sessions and issues JWTs.
  - JWTs are passed to Supabase for API requests (see `src/services/supabase.js`).
  - Application-level ownership checks supplement RLS due to Supabase free tier limits.
- **Features:**
  - Blog CRUD, status management (Draft/Published/Archived), image upload, categories, notifications.
  - Only blog owners can delete their blogs (enforced in UI and API layer).
- **Component Structure:**
  - `src/features/` contains feature modules: `authentication/`, `blogs/`.
  - `src/services/` holds API logic (Supabase, Clerk integration).
  - `src/ui/` has reusable UI components (e.g., `Modal.jsx`, `Table.jsx`).
  - `src/pages/` for route-level components.
  - `src/hooks/` for custom React hooks.

## Developer Workflows

- **Install:** `npm install`
- **Start Dev Server:** `npm run dev` (default: http://localhost:3000)
- **Environment:**
  - Set up `.env` with Supabase and Clerk keys (see `README.md` for details).
- **Database:**
  - Schema and RLS setup SQL in `README.md`.
- **Authentication:**
  - JWT template in Clerk must match claims expected by Supabase (see `README.md`).
- **Testing:** No formal test suite; manual testing via UI.
- **Styling:** Uses Styled Components for scoped styles.

## Project-Specific Patterns

- **React Query** for all data fetching/mutation (see `useBlogs.js`, `useCreateBlog.js`, etc.).
- **Ownership checks** for blog actions are done in both UI and API layer.
- **Status tags** and workflow: blogs have `draft`, `published`, `archived` states (see `StatusTag.jsx`).
- **Modal confirmation** for destructive actions (see `ConfirmDelete.jsx`).
- **Error handling:** User-facing errors via React Hot Toast.
- **Component naming:** Feature-based, e.g., `CreateBlogForm.jsx`, `useEditBlog.js`.

## Integration Points

- **Supabase:** All DB/API access via `src/services/supabase.js` and related hooks.
- **Clerk:** Auth flows in `src/features/authentication/` and `src/services/apiAuth.js`.
- **Image Upload:** Handled in blog forms, stored as URLs in DB.

## References

- See `README.md` for setup, environment, and schema details.
- See `src/services/` for integration logic.
- See `src/features/` for feature patterns.
- See `src/ui/` for UI conventions.

---

For new features, follow the feature-module pattern and use React Query for all data operations. When in doubt, check for similar patterns in `src/features/` and `src/services/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/damianr-z)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/damianr-z)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
