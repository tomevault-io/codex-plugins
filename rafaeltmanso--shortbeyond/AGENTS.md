# ShortBeyond – AI Coding Agent Guide

## Project Snapshot
- Playwright API regression suite targeting the ShortBeyond URL shortener (`shortbeyond.yaml` spins up API, web, Adminer, and PostgreSQL containers).
- Tests execute directly against the REST API; there is no frontend/browser automation.
- Repository defaults to CommonJS, but Playwright specs use ESM imports (Playwright handles this at runtime).

## Environment & Setup
- `.env` seeds runtime config: `BASE_API` (e.g. `http://localhost:3333`) plus PostgreSQL credentials; keep it in sync with the container stack.
- `playwright.config.js` reads `BASE_API` and enables `global-setup.js`, so relative service URLs resolve against the configured host.
- `global-setup.js` invokes `cleanupTestData()` (`playwright/config/database.js`) to purge users whose emails end in `@papito.dev`; use the provided factories so cleanup remains effective.
- Run `npm install` once, then `npx playwright test`; inspect HTML output with `npx playwright show-report`.

## Test Architecture
- Specs live under `playwright/e2e/` and are grouped by domain (e.g. `auth/`, `links/`, `health.spec.js`). Portuguese titles communicate expected behaviour.
- `playwright/support/fixtures.js` extends Playwright's `test` with:
  - `auth`: wraps `registerService`/`loginService` and exposes `createUser`, `login`, and `getToken` helpers.
  - `links`: pre-wired CRUD helpers from `linksService` (POST/GET/DELETE, plus `createAndReturnLinkId`).
  - `authenticatedUser`: lazily creates a fresh user via the auth fixture and returns `{ user, token }` for authenticated scenarios.
- Always import `{ test, expect }` from `support/fixtures.js` so the custom fixtures and Playwright's `expect` are consistently available.

## Support Layer Patterns
- Services (`support/services/*.js`) accept the Playwright `request` fixture and call relative endpoints (`/api/...`); avoid hardcoding the host to keep compatibility with `BASE_API`.
- Data factories (`support/factories/*.js`) centralise faker usage; `getUser()` always returns an email on the `papito.dev` domain, and `getUserWithLinks(n)` pairs one user with `n` pre-built link payloads.
- `support/utils.js` provides `generateULID()` for generating syntactically valid link IDs when exercising negative cases like DELETE 404s.

## Spec Conventions
- Use `test.beforeEach` to instantiate the relevant services with the current `request` fixture; store them in `let` bindings for reuse inside tests.
- When authentication is needed, either leverage `authenticatedUser` or manually `createUser` + `login` to capture the JWT token (`loginService` returns `{ data: { token, user } }`).
- Many negative-path tests highlight known API bugs in comments but still assert the expected behaviour (e.g. DELETE should return 404 for unknown IDs); keep those expectations unless the contract changes.
- The health check spec (`e2e/health.spec.js`) is the lone test that hits an absolute URL; new specs should prefer relative paths and the configured base URL.

## Operational Notes
- Database cleanup relies on transactional deletes; if the API schema changes, adjust `cleanupTestData()` to continue removing `papito.dev` fixtures.
- The suite runs fully parallel (`fullyParallel: true`); design new tests to avoid shared mutable state outside the database cleanup guarantees.
- ShortBeyond accepts Bearer tokens; services already attach `Authorization` headers, so pass only the raw token string.

## Need Clarification?
Call out any missing workflows (e.g. additional seeding, CI commands) so we can extend this guide.

---
> Source: [rafaeltmanso/shortbeyond](https://github.com/rafaeltmanso/shortbeyond) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-22 -->
