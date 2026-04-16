# Copilot Custom Instructions

## Project Overview

This is a coffee delivery app with a React + TypeScript frontend (Vite) and Express + TypeScript backend, managed as a pnpm workspace monorepo.

## Architecture

- `client/` — React 19 + TypeScript, Vite, react-router-dom, lucide-react
- `server/` — Express + TypeScript, in-memory data store
- Styling uses CSS variables defined in `client/src/styles/tokens.css`
- Mobile-first UI (393px viewport)

## Code Conventions

- Use TypeScript strict mode in both client and server
- React components use named exports and function declarations
- Inline styles via `Record<string, React.CSSProperties>` objects (no CSS modules)
- API routes are organized in `server/src/routes/` and mounted under `/api`
- Shared types live in `server/src/types.ts`; frontend types in `client/src/data/products.ts`
- Use pnpm, not npm or yarn

## When Reviewing Code

- Flag any use of `any` type — prefer explicit types
- Ensure API error responses include meaningful messages
- Check that new endpoints have corresponding frontend integration
- Validate that price calculations use size-based `prices` object, not a flat `price` field
- Ensure new pages include `BottomNav` if they are top-level navigation targets

## When Generating Tests

- Use Vitest for client tests, Jest for server tests
- Mock API calls in frontend tests using `vi.fn()` or `msw`
- Test Express routes with `supertest`
- Cover both happy path and error cases (404, 400)
- For components, test user interactions (click, input) not implementation details

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AlexandrSedymov) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
