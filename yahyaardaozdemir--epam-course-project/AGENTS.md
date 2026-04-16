# epam-course-project Development Guidelines

Auto-generated from all feature plans. Last updated: 2026-02-25

## Active Technologies

- TypeScript 5.x strict mode (backend + frontend) + Express 4, React 18, React Router 6, Zod, bcryptjs, jsonwebtoken, better-sqlite3, multer, Jest, Playwright (002-innovateepam-portal)

## Project Structure

```text
backend/
frontend/
tests/
```

## Commands

npm test && npm run lint

## Code Style

TypeScript 5.x strict mode (backend + frontend): Follow standard conventions

## Recent Changes

- 002-innovateepam-portal: Added TypeScript 5.x strict mode (backend + frontend) + Express 4, React 18, React Router 6, Zod, bcryptjs, jsonwebtoken, better-sqlite3, multer, Jest, Playwright

<!-- MANUAL ADDITIONS START -->
## Project-Specific Guardrails (Preserved)

- Scope authority is limited to:
	- `specs/002-innovateepam-portal/spec.md`
	- `specs/002-innovateepam-portal/plan.md`
	- `specs/002-innovateepam-portal/tasks.md`
	- `.specify/memory/constitution.md`
- Do not invent constraints, requirements, or UX behavior beyond those artifacts.
- If a conflict appears between generated guidance and the files above, follow the files above.

## Actual Monorepo Structure

```text
backend/
	src/
	tests/
frontend/
	src/
	tests/
e2e/
	tests/
specs/002-innovateepam-portal/
```

## Commands to Trust in This Repo

- `npm run dev`
- `npm test`
- `npm run test:unit`
- `npm run test:integration`
- `npm run test:coverage`
- `npm run test:e2e`
- `npm run lint`
- `npm run build`

## Implementation Safety Notes

- Roles are exactly: `submitter`, `admin`.
- Delivery order is constitution-driven: Story -> Spec -> Plan -> Tasks -> failing tests -> implementation.
- Keep strict TypeScript and exported-symbol JSDoc requirements.
- Keep changed-code coverage at least 80%.
- If a task references a missing script (for example `lint:openapi`), add it during setup tasks rather than assuming it already exists.
<!-- MANUAL ADDITIONS END -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/YahyaArdaOzdemir) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
