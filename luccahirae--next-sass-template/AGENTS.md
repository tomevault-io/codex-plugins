# Copilot Instructions

This document guides GitHub Copilot to generate code consistently for this project.

## Project Overview

- **Type:** SaaS starter template
- **Framework:** Next.js (App Router)
- **Styling:** TailwindCSS + ShadCN/UI
- **Auth:** Better Auth
- **Payments:** Stripe
- **Database/ORM:** PostgreSQL with Prisma

## File Structure

- `app/` → Next.js routing and pages
- `components/` → ShadCN and custom UI components
- `lib/` → utilities and helpers
- `hooks/` → custom React hooks

## Styling

- Use **TailwindCSS** for all styling needs
- Use **ShadCN/UI** components for consistent UI elements
- Interface should be responsive and accessible
- Follow design patterns established in existing components
- Design should be clean and modern

## Coding Conventions

- Use **functional components only**
- Use **TypeScript** with explicit prop typing (`interface Props`)
- Use **server components by default**; only use client components when necessary
- Use **path aliases** (`@/components`, `@/lib`, etc.)
- Keep formatting minimal and standard (ESLint/Prettier defaults)

## Notes for Copilot

- Prefer generating reusable UI with ShadCN components and TailwindCSS utilities
- Keep auth-related code aligned with **Better Auth** best practices
- Implement payment flows using **Stripe**
- Use **Prisma** schema and client for database access
- Use **Tanstack Query** for data fetching and caching
- Use **React Hook Form** for form handling
- Use **Zod** for schema validation
- Use **PNPM** for package management

## Features

- When implementing features in the `_features.md` file, check off the relevant boxes to indicate completion.
- Ensure that new features adhere to the project's coding conventions and structure.

---
> Source: [luccaHirae/next-sass-template](https://github.com/luccaHirae/next-sass-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-13 -->
