
# UI Change E2E Screenshot Report

When you edit files matched by this rule:

1. Follow **ui-e2e-screenshot-report** to **instruct** the operator (do not run make targets in-session).
2. End the response with the narrowest `make e2e_test_web_report_spec` or `e2e_test_management_web_report_spec` command in a fenced `bash` block (`SPEC=` covers the change).
3. Tell the operator where to open `.artifacts/e2e-reports/latest/.../index.html` after they run the command.

---
> Source: [podverse/podverse](https://github.com/podverse/podverse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-05 -->
