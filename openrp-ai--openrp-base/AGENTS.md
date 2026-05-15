## Major frameworks

- NextJS App Router for frontend and backend
- Supabase for database and authentication
- Drizzle ORM with queries for database fetching, database definition is in
  `database/`
- we are transitioning from Supabase Data API to Drizzle, so there are still
  some Supabase data fetching
- tailwindcss for styling
- shadcn for ui components (try not to modify or introduce new components under
  `components/ui`, those are reserved for shadcn. Instead just add them to the
  relevant component folder for a particular page)
- protobuf-es for app-level data. Proto definitions are under `proto/` and they
  are generated to `generated/proto`. When we fetch from the database, we
  immediately transform them into proto objects
- react-hook-form for any form building
- stepperize for multi-step components
- swr for any client-side data fetching or submission, with a universal fetcher
  in @fetchers.ts
- stripe for payment
- immer and useImmer instead of React.useState for modifying complex data states
- lucide-react for icons
- nuqs for manipulating query parameters
- git for tracking user's worlds (think of them as repos). We keep a copy of
  user's information in the database but write to both db and git
- sonner for client-side notifications

## Key directories

- app: public-facing web pages that (mostly) map directly to the URL
- app/api: public-facing server APIs, currently only consumed by the frontend in
  client components
- components: reusable React components loosely organized according to the page
  they are mainly used in
- components/ui: Shadcn components, don't add new components to this directory
- database: Drizzle database schema definitions
- lib: models/helper functions shared by both server and client components, e.g.
  database operations that are common for both client APIs and server components
  are stored under lib/api

## Api routes

- our api routes are under `app/api`
- like our pages, api routes use kebab-casing
- our apis are RESTful and should be as simple as possible
- use zod schema to parse requests
- use `parseSearchParams` from `zod-search-params` to parse search params
- api routes contain as little logic as possible and act as request/response
  parser. The main data logic should be under `lib/api`
- it should only contain routes. ALL other utilities and core business logic and
  even helper functions should be in `lib/`
- use `createSuccessResponse` and `createErrorResponse` in @api_response.ts and
  a try block, like so:

```
try {
  return createSuccessResponse(await someApi(parsedParams));
} catch (error) {
  return createErrorResponse(error);

}
```

## Style guide

- Follow Google's Typescript style guide.
- Don't use relative import paths (e.g. "../some_file"). Use absolute path (e.g.
  "@/a/b/some_file"), unless it is in the same folder.
- For file names:
  - React components should use CamelCase for convention.
  - UI components under components/ui/ should use kebab-case for Shadcn UI
    convention.
  - Web routes in app/ should use kebab-case for URL readability.
  - Everything else should be snake_case as per Google style guide.
- All APIs under app/api should be RESTful.
- ALWAYS WRITE TESTS FOR WHATEVER YOU MODIFY!!! They will save you hours of
  compute time!!!

## Testing

- test files are placed in the same folder as the source code
- test files have the suffix `.test`, so `Component.tsx` will have a test
  `Component.test.tsx`
- write vitest tests
- try to mock as little as possible - never mock unless absolutely necessary. If
  a test tests a component that references other components that we wrote, NEVER
  mock these and test real behavior.
- this includes file systems and databases. Just use node's fs with a temp
  folder and we have an in-memory pglite database in @server_api.test.helper.ts.
- remember separation of concerns. If component A includes component B, then you
  should test A is rendering B (by using data-testid) but not what B is
  rendering - that's for B's tests.
- for mocks that we change values across tests, reset them in `before()` and
  introduce properly scoped variables for them.
- put all test objects or helper/utility methods in `file.test.helper.ts`. Reuse
  existing test objects from logical places, like test users in
  `lib/api/users.test.helper.ts`.
- for database test objects, see @server_api.test.helper.ts for seeded objects
  for testing.

---
> Source: [openrp-ai/openrp-base](https://github.com/openrp-ai/openrp-base) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-14 -->
