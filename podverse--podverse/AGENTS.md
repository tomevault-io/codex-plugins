
# E2E seed `id_text` length limits

Postgres **`nano_id_v2`** is **`VARCHAR(15)`** with length **9–15** (see
`infra/k8s/base/ops/source/database/linear-migrations/app/0000_init_helpers.sql` and
`packages/orm/src/lib/nanoid.ts`).

When adding or renaming deterministic E2E fixture IDs in seed scripts or `seedConstants.ts`:

- **Count characters** on every new `id_text` before commit (descriptive names often exceed 15).
- Keep IDs in the **9–15** range; prefer existing short patterns like `e2ePodResume01` (13).
- Update **all** sync points together (seed, `seedConstants`, demo links, docs, specs).
- Run `make e2e_seed_web` (or `make e2e_seed`) to verify inserts succeed.

Full workflow: **e2e-seed-nano-id-limits** skill.

---
> Source: [podverse/podverse](https://github.com/podverse/podverse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-05 -->
