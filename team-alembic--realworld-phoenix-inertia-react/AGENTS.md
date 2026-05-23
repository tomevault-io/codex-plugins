# GitHub Copilot Instructions for RealWorld Phoenix-Inertia-React

This document provides instructions to GitHub Copilot for generating code suggestions that align with the architecture, patterns, and technologies used in this project.

## Project Overview

This is a RealWorld application implementation using:

- Elixir/Phoenix as the backend framework
- Inertia.js as the adapter between Phoenix and React
- React for frontend UI components
- TailwindCSS for styling
- Ash Framework for API modeling and domain logic

## Architecture

### Backend (Elixir/Phoenix)

- We use Phoenix 1.7+ with the latest features and patterns
- The application follows a resource-oriented approach using Ash Framework
- API definitions are in `/lib/realworld/<domain>/` with domains for accounts, articles, profile
- Controllers are in `/lib/realworld_web/controllers`
- Backend routes are defined in `/lib/realworld_web/router.ex`

### Frontend (Inertia.js + React)

- React components are in `/assets/js/pages` for Inertia page components
- Reusable components are in `/assets/js/components`
- TypeScript is used for type safety with type definitions in `/assets/js/types.ts`
- TailwindCSS is used for styling with configuration in `/assets/tailwind.config.js`

## Coding Standards

### Elixir/Phoenix

- Follow the official Elixir style guide
- Use pipe operators `|>` for function chaining when appropriate
- Keep functions small and focused on a single responsibility
- Use pattern matching instead of conditionals when possible
- Prefer explicit function calls over macros unless macros significantly improve readability
- Apply the 'single level of abstraction' principle to split functions over 30 lines into small functions

### JavaScript/React

- Use functional components with hooks
- Use TypeScript for type safety
- Follow React best practices with proper component composition
- Prefer named exports over default exports

### CSS/TailwindCSS

- Use Tailwind utility classes directly in components
- Extract common patterns to components rather than creating custom CSS classes
- Follow mobile-first responsive design

## Technologies Reference

### Ash Framework

- Used for domain modeling and API building
- Resources are defined in `/lib/realworld/resources`
- Follow Ash patterns for queries, mutations, and relationships
- use `mix ash.codegen <migration_name>` to generate database migrations, do not generate ecto migrations directly

### Inertia.js

- Bridge between Phoenix and React
- Controllers should return Inertia responses using the `render_inertia` helper
- Page components receive props from the controller

### React + TypeScript

- Use TypeScript for all components
- Define types for component props and API responses
- Use React hooks appropriately for state and side effects

## Testing

- Test files are located in the `/test` directory
- Backend tests use ExUnit
- Frontend tests use Jest and React Testing Library

## Common Patterns

### Authentication

- Authentication is handled using Ash Authentication
- User authentication flows follow standard Ash Authentication patterns

### Data Fetching

- Data is fetched on the server and passed to components via Inertia props
- Use the provided Inertia helpers for client-side navigation

### Error Handling

- Use proper error handling with meaningful error messages
- Follow the established pattern for error display in the UI

---
> Source: [team-alembic/realworld-phoenix-inertia-react](https://github.com/team-alembic/realworld-phoenix-inertia-react) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-22 -->
