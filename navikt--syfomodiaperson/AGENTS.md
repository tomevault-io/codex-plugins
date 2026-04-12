# Repository Guidelines

## Your role

- You are an AI coding assistant working with a senior developer.
- Enhance productivity through research and targeted assistance.
- The developer maintains ownership and decision-making authority.
- We are a team. Ask questions, provide options, and suggest improvements without making unilateral changes.

## Overview of the domain of this repository

- This repository is a frontend application for internal use in the Norwegian Welfare Administration (NAV).
- The application is used to manage and help reduce sick leave ("sykefravar" in Norwegian).
- The main users of the application are NAV employees ("veileder" in Norwegian) who work with sick leave cases.
- Information in the application is related to users' health and employment status.
- This application is always in the context of one user ("sykmeldt" in Norwegian) at a time.
- Information in the application is often sensitive personal data, so security and privacy are top priorities.
- This frontend application fetches data from several backend services in a microservice architecture. Those apps use
  Domain-Driven Design (DDD) principles to model the domain.
- Domain names in the app:
  - "veileder" (NAV employee helping the sick person),
  - "sykmeldt" (the sick person being helped),
  - "arbeidsgiver" (employer),
  - "sykmelding" (sick leave note from doctor),
  - "sykepengesoknad" (application for sick pay by NAV),
  - "behandler" (healthcare provider, doctor or therapist),
  - "oppfolgingsplan" (plan between employer and sick person on how to get back to work faster),
  - "dialogmote" (dialogue meeting between NAV, employer, sick person and doctor).
  - "dialogmelding" (messages between NAV and doctor).
  - "aktivitetskrav" (activity requirements for sick person to be eligible for sick pay),
  - "oppfolgingsoppgave" (smal note/task for NAV employee related to a sick person, with text and due date).
  - "enhet" (NAV office unit where the sick person is being consulted from).
- Consider every unknown English word in the codebase a domain-specific term from the Norwegian language.

## General guidelines

- Write code in English, but use Norwegian for domain-specific terms. Don't use æ, ø or å in Norwegian words. Eg.
  getSykmeldtData(), not getSickLeaveData()
- If there are more than one obvious way to solve a problem, stop and ask what approach I would like to take
- If you need more context to make an informed suggestion, ask questions until you know enough to make the right choice
- When prompted to write code, start by create a high-level plan of how to implement the feature or fix the bug. Ask for
  feedback on the plan before writing any code.
- Prefer readable code over "clever code"
- Whenever implementing a new functionality, find similar implementations in the codebase and follow those
  implementation patterns
- Refactor large functions and classes to optimize for readability and re-use
- Avoid adding external packages unless necessary
- If working on a feature that requires refactoring, try to do it up-front before adding new functionality in a separate
  commit or PR ("make the change easy, then make the easy change")
- Update this file with any new guidelines that arise during development

## Project structure

- React with TypeScript for frontend development. Entry point is `src/index.tsx`.
- Node.js with Express and TypeScript for the backend server. Entry point is `server.ts` and the `/server` folder.
- The Express-server works as a backend-for-frontend proxy which handles proxying between frontend and several backend
  apps in our microservice. The proxy also handles authentication, session management and some caching.
- Webpack is our module bundler
- Vitest is our testing framework, with React Testing Library for component tests
- We have a design system available as a dependency from `@navikt/ds-react`
- Tailwind for styling when the design system does not cover our needs
- Axios for API calls
- React Query for data fetching and caching
- React Router for routing, see `src/routers/AppRouter.tsx`
- React Context for global state management when necessary
- Unleash for feature toggles
- The main folders are:
  - `src/api/`: functions for API calls using Axios for GET, POST, etc.
  - `src/components/`: Reusable React components
  - `src/sider/`: Main routes/pages and their subcomponents
  - `src/hooks/`: Custom React hooks
  - `src/data/`: React Query data-fetching hooks and related types
  - `src/utils/`: Utility functions and helpers
  - `src/mocks/`: Mock data for development and testing
  - `test/`: Tests and test utilities

## Coding conventions

- Use functional components only.
- Prefer hooks over HOCs.
- Keep components pure; move business logic to hooks/services.
- One component per file unless tightly coupled.
- Avoid re-renders: memoize selectively, never prematurely.
- Use is/has prefix for boolean values. E.g. isActive, hasAccess
- Use function declarations over arrow functions for all named functions/components.
- Use interfaces for component props. The interface should be named `Props` only. Ask if several prop-interfaces are
  needed in the same file.
- Update existing code to match these conventions when touching it.
- No implicit `any`.
- Never use `as any` or non-null assertions without a comment explaining why.
- Prefer React Query for async server state.
- Local UI state: `useState`
- Keep global state minimal; prefer React Context only when necessary.
- Write documentation comments for all exported functions, types, and components using TSDoc format.
- Avoid inline comments, except where they add explicit value

## Testing

- Use Vitest + React Testing Library.
- Mock Service Worker (MSW) for API mocking.
- Test files with `...Test.tsx` suffix.
- Write tests for new functionality.
- Use the Arrange, Act, Assert-pattern in tests, and separate these sections with a blank line.
- Mostly test user interactions and component behavior.
- Nest tests using `describe` blocks for different scenarios.
- Update tests when touching existing code to match current behavior.
- Use `beforeEach` hooks in tests for setup logic.

## Build, Test & Development Commands

- `npm install` update dependencies.
- `npm run start` starts the development server with hot-reloading.
- `npm run test` runs all tests once.
- `npm run lint-and-typecheck` runs ESLint and TypeScript type checking.
- `npm run build` builds the frontend and backend code into the `dist/` folder.

## Limitations in legacy code

- We have `.less`-files for styling, but these are legacy and should be avoided in new code. Refactor old .less-files to
  use Tailwind or CSS modules when touching them.
- We have some components from our old design system, named `nav-frontend-*` in `package.json`. These are legacy and
  should be avoided in new code. Refactor old `nav-frontend-*` components to use `@navikt/ds-react` components when
  touching them.
- We have some js-files in the codebase, but new code should always be in TypeScript. Refactor old js-files to
  TypeScript when touching them.
- The webpack configuration is legacy and should not be changed unless absolutely necessary.
- The Express backend server is legacy and should not be changed unless absolutely necessary.

## Analytics with Umami

- We use [Umami](https://aksel.nav.no/god-praksis/artikler/male-brukeradferd-med-umami) for measuring user behavior.
- Event tracking utilities are in `src/utils/umami.ts`.
- Event types and property shapes come from `@navikt/analytics-types` — use `Events`, `TaxonomyEvent`, and the per-event `*Properties` types from that package.
- Follow the [NAV Umami taxonomy](https://startumami.ansatt.nav.no/taksonomi) for event naming.
- To track a new type of event:
  1. Check if the event already exists in `@navikt/analytics-types` (see the `Events` object).
  2. If it exists, import `Events` from `@/utils/umami` and call `trackEvent({ name: Events.MY_EVENT, properties: { ... } })` from the relevant component.
  3. If it does not exist, define a new event constant and properties type locally and add it to the `Event` union in `umami.ts`, following the shape used by `@navikt/analytics-types`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/navikt)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/navikt)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
