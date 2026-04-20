# Concurrency patterns (context, cancellation, workers)

References
- @.cursor/rules/go/go_base.mdc

Guidelines
- Use context for cancellation; select on ctx.Done().
- Keep goroutines short-lived; avoid leaks.
- Use buffered channels deliberately; document buffer rationale.

Acceptance checklist
- Examples compile; vet is clean.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/xcono) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
