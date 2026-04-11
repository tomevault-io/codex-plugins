# Octagonl Next.js App — Copilot Instructions

This is an Octagonl platform application built with Next.js. Follow these conventions when generating or modifying code.

## Project Context
- This app is part of the Octagonl platform ecosystem at `octago.nl`
- Authentication is handled by Microsoft Entra External ID (centralized SSO)
- The app deploys to Azure App Service via GitHub Actions with OIDC federation
- Platform docs are in the `docs/octagonl-docs/` git submodule

## Authentication Rules
- **Use `oid` claim, NOT `sub`** — `sub` is pairwise in Microsoft Entra (differs between app registrations). `oid` (Object ID) is stable across all Octagonl apps. Store it as `entra_oid`.
- Auth flow: OIDC Authorization Code + PKCE + client secret (server-rendered)
- Library: `next-auth` with Azure AD provider
- JWKS caching: 1-hour TTL via `jose` library. Never fetch on every request.
- Token validation: always validate issuer, audience, expiry, and signature
- Session duration: 8 hours, HTTP-only cookies

## Coding Standards
- TypeScript strict mode (`"strict": true`)
- Node.js 20 LTS, npm as package manager
- ESLint flat config with zero warnings (`--max-warnings 0`)
- Use `.js` extensions in imports (ESM)
- Prefer named exports
- Use `const` by default; `let` only when necessary
- No `any` — use `unknown` and narrow with type guards
- Validate at system boundaries with Zod

## Azure & Infrastructure
- Secrets in Azure Key Vault via Managed Identity — never hardcode secrets
- OIDC federation for CI/CD — no long-lived credentials
- Naming: `{prefix}-octagonl-{app}-{env}-{component}`
- Bicep for IaC in `infra/` directory

## Security
- Helmet for security headers (API routes)
- CORS strict per-origin allowlist
- Non-root Docker user (UID 1001)
- Dependabot for dependency updates
- Never leak stack traces in error responses

## Platform API
- Base URL from `PLATFORM_API_BASE_URL` env var
- Pass user access token as `Bearer` in `Authorization` header
- Include `X-Request-Id` for correlation
- Key endpoints: `/v1/me`, `/v1/tenants`, `/v1/me/memberships`

## File Locations
- Auth config: `src/lib/auth.ts`
- API client: `src/lib/platform-api.ts`
- Auth guard component: `src/components/auth-guard.tsx`
- Session provider: `src/components/providers.tsx`
- Brand tokens: `shared/tailwind-brand.js`
- Infra modules: `infra/modules/*.bicep`
- Integration docs: `docs/octagonl-docs/app-integration/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Galacticnaut)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Galacticnaut)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
