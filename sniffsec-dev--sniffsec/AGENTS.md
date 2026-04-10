# SniffSec Workflow
## Security Standards
- Never hardcode secrets (Stripe, AWS, etc.) or PII.
- Use `npx sniffsec` as the primary verification tool.

## Build Commands
- Run Scan: `npx sniffsec`
- Install: `npm install`

## Verification
Before completing any coding task, run the scan. If `issuesFound > 0`, the task is not complete. Fix the leaks and re-run.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sniffsec-dev)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/sniffsec-dev)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
