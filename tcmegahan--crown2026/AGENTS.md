# Crown2026 – Copilot Guardrails (MANDATORY)

You are assisting in a production Django codebase.

STRICT RULES:
- DO NOT invent base classes, mixins, or abstractions.
- DO NOT reference classes unless they already exist and are imported.
- DO NOT add models, fields, or files unless explicitly requested.
- DO NOT partially patch files when asked to replace them.
- When fixing errors, prefer FULL FILE REPLACEMENT over incremental edits.
- DO NOT add future-phase features to MVP code.
- DO NOT use placeholder names or speculative patterns.
- DO NOT introduce TimeStampedModel, AbstractBaseModel, or similar unless instructed.

MVP PHILOSOPHY:
- Minimum viable, boring, deterministic code only.
- Explicit > clever.
- Simple > abstract.
- Correct > complete.

If unsure, STOP and ASK.

This alone dramatically reduces “creative” damage.
# Crown2026 AI Coding Agent Instructions

## 🏗️ Project Architecture

- **Single workspace, Django backend is authoritative.** No sub-projects or microservices; all business logic flows through the Django backend in `backend/crown_api/`.
- **Major API:** The Director Actions API (`POST /api/director/actions/`) is the central pattern for admin actions (see [backend/crown_api/director_views.py](backend/crown_api/director_views.py) and [docs/DIRECTOR_ACTIONS_API.md](docs/DIRECTOR_ACTIONS_API.md)).
- **Gold Standard Reference:** Financial Aid/Awards code is the canonical pattern; Admissions and Finance modules clone this structure (see [docs/REFERENCE_MODULE_PATTERN.md](docs/REFERENCE_MODULE_PATTERN.md)).
- **No external dependencies, migrations, or config changes unless explicitly approved.** All dependencies must be standard or already present.

## 🚦 Critical Developer Workflows

- **Build/Run:**  
  - The venv lives at the **repo root** (`.venv/`), not inside `backend/`:
    ```
    & ".venv\Scripts\python.exe" "backend\manage.py" runserver 127.0.0.1:8000 --noreload
    ```
- **Testing:**  
  - Automated: `python test_director_actions.py`
  - Manual: `source curl_examples_director_actions.sh` (see examples in file)
- **Git Discipline:**  
  - After any change:  
    ```
    git status
    git diff --name-only
    git add <files>
    git commit -m "clear message"
    git push
    ```
  - Commit early & often, use `{type}: {description}` format (e.g., `feat: director API endpoints`).

## 📚 Project-Specific Conventions & Patterns

- **No changes to auth, roles, finance, or core Django files without explicit approval.**
- **Views and API endpoints use atomic transactions and per-award error handling.**  
  - Example: [director_actions() in backend/crown_api/director_views.py](backend/crown_api/director_views.py#L462)
- **Extensible by gold standard cloning:**  
  - New persona APIs should copy the Financial Aid pattern exactly—see [docs/REFERENCE_MODULE_PATTERN.md](docs/REFERENCE_MODULE_PATTERN.md).
- **Documentation-first:**  
  - Key guides: [README_DIRECTOR_ACTIONS.md](docs/ops/README_DIRECTOR_ACTIONS.md), [IMPLEMENTATION_SUMMARY.md](docs/status/IMPLEMENTATION_SUMMARY.md), [INTEGRATION_GUIDE.md](docs/ops/INTEGRATION_GUIDE.md), [CHECKLIST.md](CHECKLIST.md), [MASTER_SUMMARY.md](docs/status/MASTER_SUMMARY.md), [FILE_INDEX.md](docs/maps/FILE_INDEX.md).
- **Quick lookup for any task:**  
  - Find code/tests/examples in [FILE_INDEX.md](docs/maps/FILE_INDEX.md)
  - API specs/examples: [docs/DIRECTOR_ACTIONS_API.md](docs/DIRECTOR_ACTIONS_API.md)
  - Automated/manual testing: [test_director_actions.py](test_director_actions.py), [curl_examples_director_actions.sh](curl_examples_director_actions.sh)

## 🔗 Integration Points

- **Frontend integration:** Use instructions/examples from [INTEGRATION_GUIDE.md](docs/ops/INTEGRATION_GUIDE.md)
- **Cross-component communication:** Always route via Django APIs; never direct DB or file access.

## 🛡️ Guardrails

- **NEVER modify root-level files (`manage.py`, `venv`, etc.)**
- **NO speculative refactors or new dependencies.**  
- **Always verify changes in dev before pushing to prod.**
- **Do NOT add endpoints under `crown_api/api_urls.py` for modules that have their own app URLs.** Use app `urls.py` only (e.g., `financial_aid/urls.py`, `admissions/urls.py`). Legacy routes in `crown_api/api_urls.py` shadow and block new app endpoints.

---

_If any section is unclear or missing key project details, please provide feedback so this guide can be improved for future agents._

---

# Crown2026 Copilot Instructions (Audit Mode)

## Non-negotiable rules

1. DO NOT modify any files unless explicitly asked  audit tasks are READ-ONLY by default.
2. DO NOT summarize findings by inferring intent  report only what the evidence files contain.
3. DO NOT skip files from the evidence pack  every file (0014) must be reviewed and cited.
4. DO NOT add recommendations outside the scope of what the evidence shows.
5. DO NOT emit any secret values, tokens, passwords, or key material  cite only rule name and file:line, never the matched text.
6. DO NOT hallucinate URLs, SHAs, workflow names, or migration names  use only values appearing in the evidence files.
7. Cite the specific evidence file (e.g., `04_JOB_LEVEL_IF.txt line 12`) for every finding.
8. If an evidence file is missing or empty, say so explicitly  do not skip the section.

## Evidence pack format

Generated by `tools/audit/make_workspace_index.ps1` into `AUDIT_PACK_{timestamp}/`:

| File | Content |
|------|---------|
| `00_OVERVIEW.txt` | git status, HEAD SHA, last 20 commits |
| `01_TREE.txt` | Full repo file tree (excluding .git/.venv/node_modules/dist/build/coverage) |
| `02_WORKFLOWS_INDEX.txt` | List of all workflow YAML files |
| `03_WORKFLOWS_TRIGGERS.txt` | Trigger lines per workflow |
| `04_JOB_LEVEL_IF.txt` | All job-level `if:` clauses (skip-neutral hazard candidates) |
| `05_BRANCH_PROTECTION_MAIN.json` | Branch protection API response for `main` |
| `06_BACKEND_URLS.txt` | Django `show_urls` output |
| `07_MIGRATIONS.txt` | Django `showmigrations --list` output |
| `08_PY_DEPS.txt` | `pip freeze` output |
| `09_NODE_DEPS.txt` | Frontend dependency names (no lockfile content) |
| `10_SECRET_SCAN_FINDINGS.txt` | Pattern + gitleaks findings  metadata ONLY (rule/file/line) |
| `11_TRACKED_BINARIES.txt` | Tracked `.zip`, `.sqlite3`, `.db`, `.pem`, `.key` files |
| `12_UNTRACKED_ARTIFACTS.txt` | `git status --porcelain` output |
| `13_HEALTH_PROBE.txt` | Live `/api/health/` and `/api/integrity/` responses |
| `14_DEPLOY_PROD_RECENT.txt` | Recent `deploy-prod.yml` run metadata |

## Report structure (11 sections, all required)

1. **Executive Summary**  one paragraph, findings only, no recommendations
2. **HEAD SHA & Branch Status**  from `00_OVERVIEW.txt`
3. **CI/CD Integrity**  workflow triggers (`03`), job-level ifs (`04`), required checks from branch protection (`05`)
4. **Branch Protection**  full summary from `05_BRANCH_PROTECTION_MAIN.json`
5. **Backend URL & Migration Health**  from `06`, `07`
6. **Dependency Inventory**  Python (`08`) and Node (`09`)  note any obviously unusual entries
7. **Secret Scan Results**  from `10`  metadata table only; flag any non-zero hit counts
8. **Tracked Binaries & Untracked Artifacts**  from `11`, `12`
9. **Live Health Check**  from `13`  include `build_sha`, `prod_deploy_tag`, `status` fields
10. **Recent Prod Deploys**  from `14`  table of last <=20 runs with status/conclusion/sha
11. **Open Items**  list any missing evidence files, empty sections, or items requiring human follow-up

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tcmegahan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
