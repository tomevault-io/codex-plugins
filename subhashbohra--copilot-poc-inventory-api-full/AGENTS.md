# Copilot Instructions — inventory-api (POC)

## Stack and build
- Use Node.js 20 with TypeScript.
- Use Express for HTTP endpoints.
- Build with `npm run build`; tests with `npm test` (Jest with coverage).

## Architecture and coding
- This service exposes `/inventory/{sku}` used by orders-api and other consumers.
- Keep controllers thin; avoid business logic in route handlers.

## PR and process
- PR titles must start with a JIRA key.
- PR descriptions should include Impact, Risk, Rollback, and Test Evidence.

## Relationship to orders-api
- inventory-api relies on `sku` from orders-api events and contracts.
- If orders-api changes the `sku` field (name/type/nullability), this service may need:
  - updated request handling,
  - updated OpenAPI spec,
  - updated test cases and monitoring.

- When reviewing PRs here, Copilot should:
  - Highlight mismatches with the Orders API contract when evident.
  - Encourage adding or updating tests when behavior changes.

---
> Source: [subhashbohra/copilot-poc-inventory-api-full](https://github.com/subhashbohra/copilot-poc-inventory-api-full) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-26 -->
