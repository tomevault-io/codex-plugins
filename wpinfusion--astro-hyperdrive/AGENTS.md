This project uses Astro. We do use any UI frameworks like React or Vue. When suggestion a solution, try not to suggest anything that requires such a framework.

We also use Tailwind CSS for styling. When suggesting CSS, please use Tailwind CSS classes.

Astro has image components. They're called Image and Picture. They're imported from "astro:assets". Do not autocomplete the import path with something like "./Image". It will not work.

The project uses '~' as a shortcut to the src directory. For example, to import a file from src/components, you would write: '~/components/ComponentName.astro'.

Answer all questions in the style of a friendly colleague, using informal language. When I ask a follow up question that involves code, don't repeat unnecessary information. Just provide the code that requires an update.

# Tailwind CSS configuration

tailwind:
config_file: 'tailwind.config.js'
global_styles_file: 'src/styles/tailwind.css'

# Astro configuration

astro:
config_file: 'astro.config.mjs'
pages_dir: 'src/pages'
components_dir: 'src/components'
layouts_dir: 'src/layouts'

# Best practices

best_practices:

-   Ensure all components are functional and use hooks where necessary.
-   Use Tailwind CSS utility classes for styling instead of custom CSS whenever possible.
-   Keep component files small and focused on a single responsibility.
-   Use semantic HTML5 elements for better accessibility and SEO.
-   Use TypeScript for type safety.

# Additional notes

notes:

-   Comments should be added to explain complex logic.
-   Keep the project structure clean and organized.

---
> Source: [wpinfusion/astro-hyperdrive](https://github.com/wpinfusion/astro-hyperdrive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-27 -->
