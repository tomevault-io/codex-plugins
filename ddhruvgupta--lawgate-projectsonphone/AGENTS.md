# Copilot Instructions

## Code Quality
- Write clean, readable, and maintainable code
- Use design patterns and best practices appropriate for the problem at hand
- Use descriptive, intention-revealing variable and function names
- Apply OOP principles where appropriate; avoid over-engineering
- Follow TDD principles — write unit tests for all new features and bug fixes
- Consider edge cases and potential pitfalls before implementing

## Tech Stack Conventions
- Backend: C# .NET with Clean Architecture (Domain → Application → Infrastructure → API)
- Frontend: React + TypeScript + Tailwind CSS
- Follow CQRS in the Application layer — Commands and Queries handled via MediatR
- Use the repository pattern in Infrastructure; never access `DbContext` directly from controllers
- DTOs belong in the Application layer; never expose domain entities directly from the API
- Use FluentValidation for all request validation

## API & Error Handling
- All API responses follow a consistent envelope: `{ data, error, statusCode }`
- Use ProblemDetails (RFC 7807) for error responses
- Handle all exceptions in global middleware — not in controllers
- Validate inputs at the API boundary; trust data inside the domain

## Security

### Authentication & Authorisation
- JWTs must be short-lived (≤15 min access token); use refresh tokens stored in `HttpOnly`, `Secure`, `SameSite=Strict` cookies — never in `localStorage`
- Enforce `[Authorize]` on every controller/endpoint by default; opt out explicitly with `[AllowAnonymous]` only where required
- Validate the JWT `iss`, `aud`, and expiry on every request — do not accept tokens signed with `none` or HS256 with an empty key
- Implement role-based and resource-based authorisation separately: role gates what you can do, resource ownership gates what you can do it to
- Never return meaningful information in 401 vs 403 responses that could enumerate users or roles

### Input Handling & Injection Prevention
- Never log sensitive data (passwords, tokens, PII, full request bodies containing credentials)
- Always validate and sanitize every user-supplied value at the API boundary before it enters the domain
- Use parameterized queries (EF Core) — never concatenate SQL strings, LINQ expressions derived from raw input, or dynamic `ORDER BY` clauses without an allow-list
- Apply `[MaxLength]`, `[RegularExpression]`, and FluentValidation rules; reject oversized payloads at the middleware level (configure `MaxRequestBodySize`)
- Sanitize filenames and reject path traversal patterns (`../`, `..\`) before using them in file operations or blob storage keys
- Treat every value from `HttpContext.Request` (headers, query strings, route params, body) as untrusted

### Secrets & Configuration
- No secrets in source code or appsettings files committed to git — use Azure Key Vault references (`@Microsoft.KeyVault(...)`) in App Service config
- Rotate secrets on suspected compromise; prefer managed identities over connection strings wherever Azure services support it
- Never echo secrets back in API responses, logs, or error messages
- Use the `.env` / `public/config.js` pattern for frontend runtime config; never bake secrets into the Vite build bundle (`VITE_` prefixed vars are public)

### Transport & Headers
- Enforce HTTPS everywhere; reject HTTP with a 301 permanent redirect
- Set security headers on every response via `SecurityHeadersMiddleware`: `Content-Security-Policy`, `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Referrer-Policy: strict-origin-when-cross-origin`, `Permissions-Policy`
- Scope CORS to the specific SWA origin(s) — never use `AllowAnyOrigin` with `AllowCredentials`
- Enable HSTS with `includeSubDomains` and a minimum `max-age` of 1 year in production

### Rate Limiting & Abuse Prevention
- Apply rate limiting to all authentication endpoints (`/api/auth/login`, `/api/auth/register`, `/api/auth/forgot-password`) — lower limits than general API
- Return `429 Too Many Requests` with a `Retry-After` header; do not leak quota counts in error bodies
- Validate and cap pagination parameters (`pageSize` ≤ 100) to prevent large data dumps

### OWASP Top 10 Checklist
- **A01 Broken Access Control** — always verify the requesting user owns the resource before returning or mutating it
- **A02 Cryptographic Failures** — use BCrypt/Argon2 for passwords; never MD5/SHA-1; encrypt PII at rest
- **A03 Injection** — parameterized queries, HTML-encode all user content rendered in the browser
- **A05 Security Misconfiguration** — disable Swagger UI in production; remove debug endpoints; apply least-privilege IAM roles
- **A07 Identification & Authentication Failures** — enforce MFA for admin accounts; lock accounts after N failed attempts
- **A09 Security Logging & Monitoring** — log all auth events (login, logout, failed attempts, token refresh) with user ID and IP; alert on anomalies

## Frontend

### Architecture & Component Design
- Colocate component tests with the component file (`Component.test.tsx`)
- Encapsulate business logic in custom hooks; keep components purely presentational — no API calls, no business logic directly in JSX
- Prefer `const` arrow functions for all components and hooks
- Use React Query for all server state; never mirror server data into `useState` — it creates stale-state bugs
- Organise by feature, not by type: `pages/`, `components/`, `hooks/`, `services/`, `contexts/`, `types/`, `utils/`
- Keep pages thin: they compose feature components and wire up routing; business logic belongs in hooks
- Avoid prop drilling beyond 2 levels — use Context or co-located state instead

### State Management
- `useState` — ephemeral UI state (modals, form field values, toggle state)
- React Query (`useQuery` / `useMutation`) — all server/async state; do not store the result in a separate `useState`
- Context — cross-cutting concerns shared across many components (auth, theme, toast notifications)
- `useRef` — DOM references and mutable values that must not trigger re-renders (e.g. interval IDs, "already ran" flags)
- Never initialise `useState` from a URL param inside a `useEffect` — use the lazy initialiser: `useState(() => searchParams.get('x') === '1')`; use the effect only to clean up the URL

### TypeScript
- Enable `strict` mode in `tsconfig.json` — no implicit `any`, no unchecked index access
- Define explicit types for all API response shapes in `src/types/`; keep them in sync with backend DTOs
- Avoid `as` casts except at the outermost API boundary (deserialization); never use `any`
- Use discriminated unions for state machines (e.g. `| { status: 'loading' } | { status: 'error'; error: string } | { status: 'success'; data: T }`)
- Prefer `type` over `interface` for shapes; use `interface` when declaration merging is needed

### Forms & Validation
- Use `react-hook-form` + `zod` for all forms; define the schema in the same file as the form component
- Always define `path` on Zod `.refine()` errors so they bind to the correct field
- Show inline field-level errors below the input, not only as a toast — users must know which field is wrong
- Mark required fields visually; provide `autoComplete` attributes on all inputs to support password managers
- Disable the submit button while `isSubmitting` is true to prevent double-submit

### Routing & Navigation
- Use `GuestRoute` to redirect authenticated users away from `/login`, `/register`, `/forgot-password`
- Use `ProtectedRoute` (or equivalent) to redirect unauthenticated users away from app pages
- Always define a catch-all `<Route path="*">` that renders a `NotFoundPage`
- Prefer `navigate('/path', { replace: true })` after form submissions to prevent back-button resubmission
- Strip one-time URL params (e.g. `?new=1`, `?token=`) immediately after reading them with `setSearchParams({}, { replace: true })`

### Security (Frontend)
- Never store JWTs or refresh tokens in `localStorage` or `sessionStorage` — use `HttpOnly` cookies
- Never render user-supplied content with `dangerouslySetInnerHTML` unless it has been sanitized server-side through a trusted HTML sanitizer
- Ensure all external links include `rel="noopener noreferrer"` to prevent tab-napping
- All API calls must go through the centralized `apiService` (Axios instance) — do not use `fetch` directly; this ensures auth headers and error interceptors are applied uniformly
- Validate and sanitize file uploads on the client (type check, size limit) before sending — this is a UX guard only; the server must also validate
- Use `Content-Security-Policy` meta tag or header; avoid `unsafe-inline` for scripts

### Performance
- Code-split at the route level with `React.lazy` + `Suspense`; never import heavy page components eagerly in `App.tsx`
- Memoize expensive derived values with `useMemo`; memoize stable callbacks passed to child components with `useCallback` only when the child is wrapped in `React.memo`
- Avoid premature memoization — profile first with React DevTools before adding `memo`/`useMemo`
- Prefer CSS transitions/animations over JS-driven ones; use `will-change` sparingly
- Keep bundle chunks below 200 kB gzipped; use `vite-bundle-visualizer` to audit

### Testing
- Test user behaviour, not implementation: query by accessible roles/labels, not CSS class names or component internals
- Use `@testing-library/user-event` for all interactions (click, type, select) — do not use `fireEvent`
- Mock the `apiService` at the module level in tests; never make real HTTP calls
- Test error states, loading states, and empty states — not just the happy path
- Aim for coverage of all branches in custom hooks; use `renderHook` from `@testing-library/react`
- Run `npm run test:coverage` in CI; fail the build if coverage drops below the configured threshold

### Accessibility (a11y)
- All interactive elements must be keyboard-navigable and focusable
- Use semantic HTML (`<button>`, `<nav>`, `<main>`, `<section>`, `<article>`) before reaching for `<div>` with `role=`
- Every form input must have a visible `<label>` associated via `htmlFor` / `id`; never use `placeholder` as the only label
- Provide `aria-label` on icon-only buttons; use `role="alert"` on inline error messages
- Ensure colour contrast meets WCAG AA (4.5:1 for normal text, 3:1 for large text); do not rely on colour alone to convey meaning
- Test with a screen reader (NVDA/VoiceOver) before marking accessibility work as done

## Logging & Observability
- Serilog is the logging provider (configured via `UseSerilog()` in `Program.cs`)
- Inject `ILogger<T>` (from `Microsoft.Extensions.Logging`) in all services and controllers — never use `Log.*` static calls outside of `Program.cs`
- Use structured logging with named properties: `_logger.LogInformation("Fetched {Count} documents for user {UserId}", count, userId)`
- Include correlation IDs on all log entries
- Log levels: `LogDebug` for dev detail, `LogInformation` for significant events, `LogWarning` for recoverable issues, `LogError` for failures with exceptions
- Never log full exception stack traces at `Information` level — use `LogError(ex, "message")`

## Git & Branching
- Branch naming: `feature/<issue-number>-short-description`, `fix/<issue-number>-short-description`, `refactor/<issue-number>-short-description`
- Never commit directly to `main` or `develop`
- Squash commits before merging to `main`
- Every commit message must reference the issue number (e.g., `Fix login error #123`)

## Issue & Documentation Tracking
- Use GitHub Issues to track all bugs, features, and refactors
- For each issue, create a folder: `docs/issue-<number>-short-description/` with a `.md` file covering:
  - Problem description, steps to reproduce, expected vs actual behaviour, relevant logs
- For each feature, create a folder: `docs/feature-<name>/` with a `.md` file covering:
  - Feature description, user stories, design mockups, implementation trade-offs
- For each refactor, create a folder: `docs/refactor-<name>/` with a `.md` file covering:
  - Motivation, affected areas, risks, and test/validation plan
- Link documentation files to their corresponding GitHub Issues

## PR Process & Human-in-the-Loop
- Every PR must reference a GitHub Issue
- **Before opening a PR, always pause and ask the user to review the planned changes** — do not open PRs autonomously
- PR description must include: what changed, why it was changed, and how to test it
- Do not suggest merging a PR — the user decides when to merge
- Never push to `main` directly; always go through a PR
- Do not merge with failing tests or lint errors
- After implementation is complete, summarise what was done and prompt the user to review before any git push or PR creation
- For destructive operations (migrations, schema changes, config changes), explicitly flag them in the PR description and wait for user confirmation

## AI Context Files
- The `docs/ai-context/` folder contains deep project context for each layer — read the relevant file before working on a task:
  - `docs/ai-context/main.md` — overall architecture, tech decisions, project goals
  - `docs/ai-context/backend.md` — .NET API structure, patterns, critical files
  - `docs/ai-context/frontend.md` — React/TypeScript setup, component patterns
  - `docs/ai-context/database.md` — schema, EF migrations, seeding strategy
- Keep these files up to date as the project evolves — they are the source of truth for AI agents working on this codebase
- For task-specific human-readable reference, consult the relevant doc in `docs/`:
  - `docs/architecture/` — system design, backend, frontend, database, schema changelog
  - `docs/guides/` — quick start, environment setup, Docker, deployment, GitHub setup
  - `docs/testing/` — test suites, how to run tests, test results
  - `docs/tracking/implementation-checklist.md` — current phase and what's done
  - `docs/index.md` — full documentation map

---
> Source: [ddhruvgupta/Lawgate-ProjectsOnPhone](https://github.com/ddhruvgupta/Lawgate-ProjectsOnPhone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-18 -->
