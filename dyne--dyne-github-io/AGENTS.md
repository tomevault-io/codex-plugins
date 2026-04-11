# Dyne.org Website

This is the source code for the website visible at [dyne.org](https://dyne.org).

## Tech Stack

*   **Framework:** [Astro](https://astro.build/)
*   **UI Components:** Svelte, Astro components
*   **Styling:** [Tailwind CSS](https://tailwindcss.com/)
*   **Linting:** ESLint
*   **Formatting:** Prettier

## Project Structure

The project follows a standard Astro project structure:

*   `src/pages/`: Contains the pages of the website. Each `.astro` or `.md` file in this directory becomes a page on the site.
*   `src/components/`: Contains reusable UI components (`.astro` and `.svelte` files).
*   `src/layouts/`: Contains layout components that define the structure of pages.
*   `src/content/`: Contains content collections, managed by Astro's content collections API.
*   `public/`: Contains static assets that are copied to the build output as-is.
*   `astro.config.mjs`: The Astro configuration file.
*   `package.json`: Defines the project's dependencies and scripts.

## Development

To run the website locally, use the following command:

```bash
pnpm dev
```

To build the website for production, use:

```bash
pnpm build
```

## How to Contribute

When adding new features or fixing bugs, please follow the existing patterns:

*   For new pages, create a file in `src/pages/`.
*   For new reusable UI elements, create a component in `src/components/`.
*   Ensure that new code is properly linted and formatted by running `pnpm lint` and `pnpm format`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dyne)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/dyne)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
