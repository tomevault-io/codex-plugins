# General guidelines for Helsa repository

## Purpose
The purpose of this document is to provide clear guidelines for creating and editing code within the Helsa repository. Adhering to these guidelines will help maintain code quality, ensure consistency, and facilitate collaboration among team members.

## General Guidelines
- Produce clear, readable React and TypeScript code using the latest stable versions of TypeScript, JavaScript, React, Node.js, Next.js App Router, Shadcn UI, and Tailwind CSS.
- Always implement all requested features fully; do not leave out code.

## Naming Conventions

- For components use kebab-case.
- For files and folders use kebab-case.
- For classes and types use PascalCase.
- For variables, functions, and methods use camelCase.

## Folder Structure
- The folder its a monorepo structure with the following main directories:
  - `apps/`: Contains the main applications (web, native, site).
  - `packages/`: Contains shared packages and libraries (api, auth, db, ui, etc.).
  - `docs/`: Contains documentation and architecture diagrams.

### Project Structure Example
```
helsa/
├── src/
│   ├── apps/
│   │   ├── native/          # Expo/React Native client
│   │   ├── site/            # Marketing site (Next.js)
│   │   └── web/             # Core web application (Next.js)
│   ├── packages/
│   │   ├── api/             # API layer and routers
│   │   ├── auth/            # Better-Auth helpers
│   │   ├── db/              # Drizzle schema and migrations
│   │   ├── agents/          # AI/agent utilities
│   │   ├── emails/          # Transactional email templates
│   │   ├── ui/              # Shared UI kit
│   │   └── workflows/       # Workflow orchestration helpers
└── docs/                # Architecture diagrams and references
```

## TypeScript Usage
- Use TypeScript's features for type safety.
- Prefer types over interfaces for object shapes.
- Use generics for reusable components and functions.
- Enforce strict typing; avoid 'any' type.

## UI & Styling
- Use Tailwind CSS utility classes for styling.
- Follow Shadcn UI component guidelines and best practices.
- Ensure UI is responsive and accessible.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Duccem)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Duccem)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
