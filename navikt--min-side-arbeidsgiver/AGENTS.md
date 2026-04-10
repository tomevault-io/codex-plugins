# Copilot Instructions for min-side-arbeidsgiver

## Project Overview

This is the frontend and BFF for NAV's "Min side arbeidsgiver" application.
It gives employers access to NAV services, cases, notifications, organization data,
Altinn-based access control, and company information in one place.

The codebase mixes Norwegian domain terminology with English technical terms.
Keep that convention in generated code, names, and comments. Prefer existing domain words
such as `arbeidsgiver`, `virksomhet`, `tilganger`, `sykmeldte`, `saksoversikt`, and `organisasjon`.

## Tech Stack

- **Frontend:** React 19, TypeScript, Vite 6
- **Routing:** React Router DOM 7
- **REST fetching:** SWR
- **GraphQL:** Apollo Client
- **Validation:** Zod
- **UI:** NAV Aksel Design System (`@navikt/ds-react`)
- **Backend-for-Frontend:** Express 5 on Node.js
- **Testing:** Vitest, Testing Library, MSW, jest-axe
- **Package manager:** pnpm
- **Deployment:** NAIS

## Architecture & Patterns

### Frontend / BFF split

The app consists of:
- a React SPA in `src/`
- an Express-based BFF in `server/`

In local development, Vite runs on port `3000` and proxies through the BFF on port `8080`.
Do not bypass the BFF for authenticated backend calls unless the existing code already does so.

### Route organization

Top-level page setup lives in `src/Pages/Pages.tsx`.
Feature pages are organized under `src/Pages/`, for example:
- `Hovedside/`
- `Saksoversikt/`
- `OmVirksomheten/`
- `ManglerTilganger/`

Follow the existing directory structure rather than introducing a new feature layout.

### State management

This project uses React Context, not Redux/Zustand.
Important contexts include:
- `OrganisasjonerOgTilgangerContext`
- `OrganisasjonsDetaljerContext`
- `AlertContext`
- `ConsentContext`

When adding shared state, prefer extending an existing context if the domain fits.
Do not introduce a separate global state library.

### Data fetching

Use:
- **SWR** for REST endpoints
- **Apollo Client** for GraphQL-backed notifications and case data
- **Zod** for runtime validation of REST responses

If a response shape changes, update the Zod schema close to the fetch hook.
Do not rely on unchecked `any` data from API responses.

### Altinn access model

Altinn data is modeled in the `userInfo` flow and made available through
`OrganisasjonerOgTilgangerContext`.

If you add new Altinn-derived fields:
- parse them in `src/hooks/useUserInfo.ts`
- propagate them through `src/Pages/OrganisasjonerOgTilgangerContext.ts`
- preserve them when organization trees are merged or flattened

Avoid adding duplicate ad hoc access logic in page components when the shared context is the right place.

### Organization tree handling

Organization structures from Altinn and DigiSyfo are merged in shared utilities/context logic.
Be careful when changing `Organisasjon` or `OrganisasjonInfo` shapes:
- merged trees must keep the necessary fields
- flattened structures must still work
- selected organization behavior must remain stable

### Demo mode and mocks

The app supports demo mode through Vite `--mode demo` and MSW.
Mocks live in `src/mocks/`.

When changing API contracts used by the frontend:
- update mock scenarios
- update MSW handlers if needed
- keep demo mode working

### BFF conventions

The `server/` app is a thin Express BFF for authentication, proxying, and environment injection.
Keep it thin. Do not move frontend state or business-heavy feature logic into the BFF unless the
existing architecture already requires it.

Environment values are injected into `window.environment` by the server. Do not introduce `.env`
files or frontend-only environment schemes that bypass this pattern.

## Testing Patterns

- Use **Vitest** for unit/integration tests
- Use **Testing Library** for component tests
- Use **MSW** to mock HTTP requests
- Use **jest-axe** for accessibility checks

Keep tests close to the code when that is already the local pattern.

When adding or changing API fields:
- update relevant tests
- update mocks/scenarios
- prefer verifying behavior through public hooks/components rather than implementation details

## Code Style Conventions

- Functional React components only
- TypeScript first; avoid `any`
- PascalCase for components
- camelCase for hooks and utilities
- Context definitions in `*Context.ts`
- Providers in `*Provider.tsx`

Prefer extending existing utilities and patterns over introducing parallel abstractions.
Use concise comments only when the code would otherwise be hard to follow.

## UI Guidance

This app uses NAV Aksel components and established patterns.
Preserve the existing visual language and accessibility standards.
Do not introduce a custom design system or one-off styling conventions if Aksel already covers the use case.

## GraphQL

GraphQL types are generated. Do not manually edit:
- `src/api/graphql-types.ts`

If GraphQL operations change, regenerate types with:
- `pnpm run gql:generate`

## Environment Configuration

Relevant runtime configuration comes from the Express server and NAIS config.
Check:
- `server/server.js`
- `nais/*.yaml`

Do not hardcode environment-specific URLs when an existing helper or config path already exists.

## Useful Commands

```bash
pnpm dev
(cd server && npm start)
pnpm test
pnpm lint
pnpm build
pnpm run gql:generate
```

## What NOT to Do

- Do not introduce Redux, Zustand, MobX, or another global state library
- Do not replace SWR/Apollo with a new data fetching layer
- Do not manually edit generated GraphQL types
- Do not bypass Zod parsing for REST responses that are already validated
- Do not bypass the BFF for authenticated NAV backend traffic without a clear existing pattern
- Do not introduce `.env`-driven frontend config as a parallel system
- Do not replace Aksel components with custom UI primitives without a strong reason
- Do not break demo mode when API contracts change

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/navikt)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/navikt)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
