# GitHub Copilot Instructions

## Project Overview

This is a web application built with Next.js, React, Tailwind CSS, and TypeScript. The project is configured with strict ESLint rules and Prettier for code quality and consistency. Please adhere to the following technologies, versions, and best practices when providing assistance.

## Core Technologies

- **Next.js**: We are using **Next.js 15**.
  - Utilize the **App Router** for routing and layouts.
  - Employ **React Server Components (RSCs)** for data fetching and server-side logic.
  - Use **Server Actions** for mutations and form submissions.
- **React**: We are using **React 19**.
  - Leverage new features like Actions, and use Hooks where appropriate.
- **Tailwind CSS**: We are using **Tailwind CSS v4**.
  - Follow a **utility-first** approach.
  - Adhere to the project's `tailwind.config.js` for theme customizations.
  - The `prettier-plugin-tailwindcss` is used to automatically sort classes.
- **TypeScript**: We are using TypeScript with **strict mode enabled**.
  - **Strictly no `any` type**. This is enforced by our ESLint configuration (`@typescript-eslint/no-explicit-any`). Use specific types or `unknown` with type guards.
  - All functions and variables must have explicit types.

## Best Practices

### General

- Follow **SOLID principles** for clean and maintainable code.
- Write **self-documenting code** with clear names for variables, functions, and components.
- Keep functions and components small and focused on a single responsibility.
- Adhere to the ESLint and Prettier configurations in the repository.

### Next.js

- Prefer **Static Site Generation (SSG)** where possible for performance.
- Use **Server-Side Rendering (SSR)** for pages with highly dynamic data.
- Use Image Component for Next.js for optimized image loading.
- Use **Link Component** for navigation to ensure client-side transitions.
- Use "@/app/" for absolute imports to simplify import paths.

### Tailwind CSS

- Adopt a **mobile-first** design approach.
- Organize utility classes in a consistent order (our Prettier plugin handles this automatically).
- **Do not use inline colors** (e.g., `style={{ color: '#ff0000' }}` or `text-[#ff0000]`). Instead, define color tokens in the global CSS (e.g., using CSS variables or Tailwind theme extensions) and reference them via Tailwind utility classes.
- Variables defined are:
  --color-action-gradient-[50|100|200|300|400|500|600|700|800|900]
  --color-neutral-[50|100|200|300|400|500|600|700|800|900]
  --color-purple-[50|100|200|300|400|500|600|700|800|900]
  --color-blue-[50|100|200|300|400|500|600|700|800|900]
  --color-teal-[50|100|200|300|400|500|600|700|800|900]
  --color-yellow-[50|100|200|300|400|500|600|700|800|900]
  --color-orange-[50|100|200|300|400|500|600|700|800|900]
  --font-mono
  --font-work-sans
  --font-roboto-mono
  --font-darker-grotesque
  --text-tiny
  --text-display-[lg|sm]
  --text-web-[h1|h2|h3|h4|h5|h6]
  --text-product-[h1|h2|h3|h4|h5|h6]
  --text-para-[lg|md|sm|xs]
  --text-label-[lg|md|sm|xs]
  --text-overline-[sm|xs]
  --text-underline-[lg|md|sm]
  --leading-display-[lg|sm]
  --leading-web-[h1|h2|h3|h4|h5|h6]
  --leading-product-[h1|h2|h3|h4|h5|h6]
  --leading-para-[lg|md|sm|xs]
  --leading-label-[lg|md|sm|xs]
  --leading-overline-[sm|xs]
  --leading-underline-[lg|md|sm]
  --tracking-tight
  --tracking-normal
- use them as tailwind classes, e.g., `text-blue-500`, `bg-neutral-100`, in case of gradient use might have to use text-[var(--color-action-gradient-500)].

### TypeScript

- Use **utility types** like `Partial`, `Required`, and `Omit` to create precise type definitions.
- Separate type definitions into their own files (e.g., `types/` directory) for better organization.

## Code Style

- Use **functional components** and **React Hooks**.
- Use **arrow functions** for component definitions.
- Use **single quotes** for strings.
- Indentation: **2 spaces**.

## Icons

- Use Icons component in app/\_components/Icons.tsx
- It uses flat icons, so you can use the `name` prop to specify the icon name. e.g., `<FiIcon name="home" />`.

## Typography

- Use the `Paragraph`, `Heading`, `Label` component from `app/_components/Typography.tsx` for consistent typography.

## Buttons

- Use the `Button` component from `app/_components/Button.tsx` for consistent button styles.

---
> Source: [AYUSHSAINI9876/HelloIvy_Frontend](https://github.com/AYUSHSAINI9876/HelloIvy_Frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-01 -->
