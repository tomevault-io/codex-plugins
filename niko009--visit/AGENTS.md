Project Guidelines: Content-Driven Astro
========================================

This is an **Astro 5.x** project using **Git-based CMS (TinaCMS)** and **Content Collections** for type-safe, static content management.

Tech Stack
----------

-   **Framework**: Astro 5.x

-   **CMS**: TinaCMS (Git-based)

-   **Data Layer**: Astro Content Collections (Zod-validated)

-   **Language**: TypeScript

-   **Styling**: Tailwind CSS

-   **Interactivity**: React components (`client:*` directives)

-   **Rendering**: Static Site Generation (SSG) by default

Architecture & Data Flow
------------------------

### The Source of Truth

1.  **TinaCMS (UI)** saves changes as JSON/Markdown files in `src/content/`.

2.  **Astro (Build)** reads these files using `getCollection()` or `getEntry()`.

3.  **Zod (Schema)** ensures every piece of content matches the defined interface.

### Project Structure

Plaintext

```
src/
├── components/     # Static Astro UI components
├── content/        # Content Collections (Data source)
│   ├── config.ts   # Zod schema definitions
│   └── services/   # MD/JSON content files
├── react/          # Interactive React components
├── layouts/        # Page layouts
├── pages/          # File-based routing
└── tina/           # TinaCMS configuration & schemas

```

Content Guidelines
------------------

### Content Collections (Crucial)

-   **Always** define schemas in `src/content/config.ts`.

-   Use Zod for runtime validation of content structure.

-   Never hardcode content in `.astro` files if it needs to be managed via CMS.

-   Example:

    TypeScript

    ```
    // src/content/config.ts
    export const collections = {
      services: defineCollection({ /* schema */ }),
    };

    ```

### TinaCMS Schema

-   Keep `tina/config.ts` synced with `src/content/config.ts`.

-   Map TinaCMS fields to the corresponding Content Collection schema.

Development Standards
---------------------

### Astro Components

-   Fetch data only via `getCollection()` in the frontmatter.

-   Use `Astro.props` to pass data to child components.

-   Example:

    Фрагмент кода

    ```
    ---
    import { getCollection } from 'astro:content';
    const services = await getCollection('services');
    ---
    {services.map(s => <div>{s.data.title}</div>)}

    ```

### React Components

-   Use for interactive elements only (modals, complex forms, filters).

-   Pass static data from Astro as props.

-   Keep components "dumb" (presentation-only); business logic lives in Astro frontmatter.

### SSG Optimization

-   Build time is the only time data fetching happens.

-   Avoid external API calls during runtime.

-   If dynamic data is needed (e.g., real-time form submission), use standard serverless API routes (`src/pages/api/*.ts`).

Configuration & Build
---------------------

### Commands

-   `npm run dev`: Start Astro dev server.

-   `npx tinacms dev`: Start the TinaCMS local preview (runs alongside dev).

-   `npm run build`: Static build (triggers SSG process).

### TypeScript Best Practices

-   Use `CollectionEntry<'collectionName'>` to type-check your content objects.

-   Strict mode is required.

Conventions
-----------

### Import Statements

-   Use path aliases: `@/components/`, `@/content/`, `@/react/`.

### File Naming

-   Collections: `kebab-case` folders (e.g., `src/content/services/`).

-   Components: `PascalCase`.

-   Pages: `kebab-case`.

### Security

-   **No API Keys** in client-side code.

-   Validate all user-submitted data (from forms) using Zod on the server side.

-   Git is your security layer; content history is maintained via commits.

* * * * *

**Note**: When adding new features, always ask: *"Does this belong in a Content Collection?"*. If the data is managed by the client, it **must** be a Content Collection.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/niko009) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
