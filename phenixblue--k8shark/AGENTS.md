
# Run GolangCI-Lint Before Push

Before any `git push` to a branch used for a PR:

1. Install the pinned CI lint version:
   - `go install github.com/golangci/golangci-lint/cmd/golangci-lint@v1.64.8`
2. Run the same lint command as CI from repo root:
   - `golangci-lint run`

Rules:

- Do not push if lint fails.
- Use the same execution format as CI (no extra flags).
- Report lint result before pushing (pass/fail and any fixes applied).
- If local PATH/version mismatch prevents lint execution, resolve that first, then rerun lint before pushing.

---
> Source: [phenixblue/k8shark](https://github.com/phenixblue/k8shark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-26 -->
