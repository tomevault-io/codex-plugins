# AI Coding Agent Guidelines for `my-website`

Welcome to the `my-website` project! This document provides essential guidelines for AI coding agents to be productive and aligned with the project's conventions and architecture.

## Project Overview
- **Framework**: This is a [Next.js](https://nextjs.org) project using the App Router.
- **Styling**: Tailwind CSS is used for styling, with dark mode support enabled.
- **Database**: Supabase is used for data storage and authentication.
- **Key Features**:
  - Dynamic filtering and search for listings.
  - Client-side and server-side rendering.
  - Integration with external services like WhatsApp and Vercel Analytics.

## Codebase Structure
- **`app/`**: Contains pages and layouts. Follow the App Router conventions:
  - `page.tsx`: Defines the main content for a route.
  - `layout.tsx`: Defines shared layouts for routes.
- **`components/`**: Reusable UI components (e.g., `Navbar`, `Footer`, `FilterSidebar`).
- **`lib/`**: Utility functions, types, and Supabase client setup.
- **`hooks/`**: Custom React hooks (e.g., `useFuzzyCazari`).
- **`public/`**: Static assets like images.

## Development Workflow
### Running the Project
- Start the development server:
  ```bash
  npm run dev
  ```
- Open [http://localhost:3000](http://localhost:3000) to view the app.

### Building the Project
- Build for production:
  ```bash
  npm run build
  ```

### Linting
- Run ESLint to check for code quality:
  ```bash
  npm run lint
  ```

## Project-Specific Conventions
### Client vs Server Components
- Default to **Server Components**. Use `"use client"` only when necessary (e.g., for hooks or browser-only APIs).

### Styling
- Use Tailwind CSS classes for styling.
- Dark mode is enabled via the `next-themes` package.

### Routing
- Pages are located under `app/**/page.tsx`.
- Use `next/link` for internal navigation and `next/image` for optimized images.

### TypeScript
- Strongly type all props and return values.
- Use types defined in `lib/types.ts` where applicable.

### API Integration
- Use the Supabase client (`lib/supabaseClient.ts`) for database interactions.
- Environment variables for Supabase are required:
  - `NEXT_PUBLIC_SUPABASE_URL`
  - `NEXT_PUBLIC_SUPABASE_ANON_KEY`

## Examples of Common Patterns
### Dynamic Filtering
- The `useFuzzyCazari` hook in `hooks/useFuzzyCazari.ts` is used for filtering listings based on user input.

### Middleware
- The `middleware.ts` file handles maintenance mode redirection based on the `NEXT_PUBLIC_MAINTENANCE_MODE` environment variable.

### Utility Functions
- Use `slugify` from `lib/utils.ts` to generate URL-friendly slugs.

## External Integrations
- **WhatsApp**: The `WhatsAppButton` component generates a link to contact property owners.
- **Vercel Analytics**: Integrated via `@vercel/analytics`.

## Contribution Guidelines
- Keep diffs minimal and focused.
- Follow existing code style and conventions.
- Add new imports at the top of the file without reordering unrelated imports.

## Notes for AI Agents
- Always fetch the latest file content before making edits.
- Use concise, focused diffs with placeholders for unchanged code.
- Avoid introducing unnecessary dependencies or reformatting unrelated code.

For any questions or clarifications, refer to the user for guidance.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/florinasiminei)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/florinasiminei)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
