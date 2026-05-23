# Quiddly Organization — Global Copilot Rules

## PR Reviews
- When doing a Review on a Pull Request, also add a comment with a 5-8 sentences long suggestion for Release Note for the feature. Specifically add if any changes have been made to the Rights that are needed in the API and explain what the changes do.

## Workspace Structure
This is a multi-repo workspace for the **Finq platform** (fintech/factoring):
- `finq.api` — Python (Flask-RESTx) backend API, Celery workers, PostgreSQL, SQLAlchemy ORM.
- `finq.admin-ui` — React 18 + Backbone.js admin portal (data-dense back-office).
- `finq.client-ui` — React 18 client portal (factoring/financing clients).
- `finq.debtor-ui` — React 17 + Backbone.js debtor portal (sensitive financial/payment UI — use empathetic messaging).
- `finq.sendgrid-templates` — SendGrid email templates.
- `finq.query_report` — Query report definitions deployed to environments.

## Cross-Repo Conventions

### Branching & Commits
- Release branches follow `release/v1.XXX` naming.
- Feature branches reference Jira tickets: `QDLY-XXXX_description` or `QDLY-XXXX-description`.
- Commit messages should reference the Jira ticket: `QDLY-XXXX: Description`.

### Multi-Repo Feature Workflow
When implementing a feature that spans API + UI:
1. Define new rights in `finq.api/finq/api/rights.py` (with Jira ticket comment).
2. Add API endpoints/models in `finq.api`.
3. Add the corresponding `PERMISSIONS` constant in the UI's `src/utils/constants.js`.
4. Update TypeScript types in the UI's `src/types/` before using new fields in components.
5. Implement the UI feature.

### Rights Flow (Critical)
- API: Define in `finq/api/rights.py` as `Rights.SOME_RIGHT = 'SomeRight'`.
- API: Use via `@requires_auth(rights=[Rights.SOME_RIGHT])`.
- Admin UI: Add to `PERMISSIONS` in `src/utils/constants.js`, check via `useUser` hook.
- Client UI: Same pattern with `PERMISSIONS`.
- Always keep API and UI rights constants in sync.

### Financial Data
- Monetary values: always pair `amount` + `amount_locale` (system-currency equivalent).
- Use `fields.Float()` in API serializers for money; `AdvancedTableMoneyProps` in UI columns.
- `payment_transaction.amount` = original stored amount (immutable).
- `payment_transaction.balance` = computed: `amount - links - returns - write_offs - revaluations`.

### Constants & Enums
- API: All domain constants in `finq/common/constants.py` as `ValuesMixin` classes.
- UI: Domain constants in `src/constants/` — never use bare strings.

### Database Safety
- **STOP AND ASK** before executing any SQL query, mutation, or migration. Show the SQL for review first.
- Alembic migrations run inside Docker: `dev/db_create_migration.sh`, `dev/db_upgrade.sh`.
- SQL views are managed outside Alembic — update the `.sql` file only.

### Language & Dates
- UI: Use **dayjs** for new code. Never introduce new `moment` usage.
- i18n: All user-facing strings must use `i18next` translation keys.
- API: Use Swedish locale (`sv`) not `se` (which is the country code, not language code).

---
> Source: [quiddly-com/.github](https://github.com/quiddly-com/.github) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-23 -->
