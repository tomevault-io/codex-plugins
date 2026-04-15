# Azure Lab Guidelines

Concise contract for creating consistent, secure Azure labs in this repo.

## 1. Directory Layout (required)

```
labs/
  <service-name>/
    README.md
    infrastructure/
      main.bicep
      parameters.example.json
      parameters.json        # optional (never commit secrets)
    scripts/
      deploy.sh
      validate.sh
      cleanup.sh
    tests/ (optional)
      smoke-test.sh
```

Rules:
- Folder and file names: lowercase, hyphens (e.g. `azure-vm-lab`).
- Bicep lives in `infrastructure/`; helper scripts in `scripts/`.
- Keep labs self-contained (no cross-lab dependencies).

## 2. README Requirements

Every lab `README.md` MUST include (in order):
1. Overview (1–3 short paragraphs)
2. Prerequisites (CLI tools, roles, quotas)
3. Deployment (one-liner + expanded commands)
4. Validation (steps + success criteria)
5. Cleanup (tear-down commands)
6. (Optional) Troubleshooting (max 3–4 common issues)

Formatting:
- Use fenced code blocks, one command per line.
- Reference current relative paths (assume user is inside `labs/<lab-name>/`).
- Explain any `.env.example` usage briefly.

## 3. Bicep Templates

- Primary template: `infrastructure/main.bicep`
- Provide `parameters.example.json` (no secrets, sensible sample values).
- Comment non-obvious parameters.
- Export critical outputs (`endpoints`, `resourceIds`, etc.).
- Naming patterns:
  - Resource group: `rg-<service>-lab-<shortid>`
  - Storage account: `st<shortunique>`
- Prefer short, deterministic suffixes (avoid randomness that hinders repeatability).

## 4. Scripts

Required bash scripts in `scripts/`:
- `deploy.sh` (idempotent provisioning)
- `validate.sh` (prints PASS/FAIL; exits non-zero on failure)
- `cleanup.sh` (idempotent teardown; safe if rerun)

Guidelines:
- `set -euo pipefail`
- Clear log messages (`[INFO]`, `[ERROR]`, `[PASS]`)
- Parameterize (env vars or flags) rather than hard-coding values.
- Never echo secrets.

## 5. Validation & (Optional) Tests

- `validate.sh` should check key resources (existence, basic readiness).
- Optional `tests/smoke-test.sh` for a deeper functional check.
- Keep tests fast (<1–2 minutes).

## 6. Security & Secrets

- Do NOT commit secrets, tokens, certificates, or real subscription IDs.
- Use placeholders (e.g. `REPLACE_ME_CLIENT_ID`).
- Document required env vars in README and provide `.env.example`.
- Ensure `.env` is in `.gitignore`.

## 7. CI / PR Checklist (Suggested)

Include in review (manual or workflow):
```
az bicep build --file infrastructure/main.bicep
scripts/validate.sh
gitleaks detect --source .
```
Verify:
- README sections present.
- No secrets / `.env` not committed.
- `parameters.example.json` exists and is sanitized.

## 8. README Authoring: DO / AVOID

DO:
- Assume fresh clone, user starts in lab root.
- Provide deploy → validate → cleanup flow.
- Keep prose minimal; prioritize actionable commands.
- Use consistent resource naming per contract.
- Show how to set env vars safely.

AVOID:
- Long historical/migration narratives.
- Repeating identical commands in multiple sections.
- Architecture deep dives (move those to separate docs if needed).
- Embedding secrets or suggesting committing them.
- Unnecessary warnings already mitigated by patterns here.

## 9. Updating an Existing Lab

Before modifying a lab:
1. Run `deploy.sh` then `validate.sh` (ensure still works).
2. Adjust README paths if structure changes.
3. Re-check outputs exported by Bicep after edits.
4. Trim any stale troubleshooting entries.

## 10. Quick Starter Template (Optional Copy/Paste)

```
# <Lab Name> Lab

## Overview
<Brief purpose>

## Prerequisites
- Azure CLI
- Role: <e.g. Contributor>
- Quota: <if needed>

## Deployment
```bash
az group create --name rg-<service>-lab-<id> --location <region>
./scripts/deploy.sh
```

## Validation
```bash
./scripts/validate.sh
```
Expected: PASS and printed endpoint.

## Cleanup
```bash
./scripts/cleanup.sh
```
```

---

By following these rules, contributed labs remain consistent, secure, and easy to validate.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/acworkma) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
