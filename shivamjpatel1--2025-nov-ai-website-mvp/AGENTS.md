# Copilot Instructions for Setu Repository

## Authoritative Baseline

**Current Stable Baseline:**
- **Tag:** `setu-repo-stable-2026-03-06`
- **Commit:** `389e83b`
- **Branches:** `restore/correct-frontend`, `backup/repo-stable-2026-03-06`, `chore/repo-hardening-and-legacy-cleanup`

The active frontend source of truth is the current stable repository state on the `restore/correct-frontend` lineage. This represents the working, production-ready implementation recovered and stabilized in March 2026.

## Critical Do-Nots

### 1. Do NOT Reintroduce Legacy/Archive Code

Do not suggest or reintroduce code from:
- Any paths containing `archive/`, `legacy/`, `backup/`, `rescue/`, or `old/`
- Branches with names like `backup/*`, `rescue/*`, `dump/*`, or recovery timestamps
- Files marked as debug-only harnesses (e.g., `frontend/pages/admin/cms/debug/`)
- Documentation from `docs/archived/` or historical phase documentation

**Why:** The repository has gone through multiple recovery attempts. Many branches and files represent failed states or experimental work that should not be revived.

### 2. Do NOT Mix Backend and Frontend Work

**Frontend-only work** should stay in:
- `frontend/pages/`
- `frontend/components/`
- `frontend/contexts/`
- `frontend/hooks/`
- `frontend/lib/`
- `frontend/styles/`

**Backend-only work** should stay in:
- `backend/app/`
- `backend/tests/`
- `backend/alembic/`
- `backend/scripts/`

**Exception:** If a feature genuinely requires changes to both frontend and backend (e.g., new API endpoint + UI), document why in the PR description. When practical, split into two PRs: backend API first, then frontend consumption.

**Why:** Mixed-scope PRs make code review difficult and can accidentally drag in unrelated changes. Backend migrations should not trigger frontend refactors, and vice versa.

### 3. Do NOT Casually Revive Debug or Unverified Components

Some components and files are currently under review as deletion candidates:
- Debug pages (e.g., `frontend/pages/admin/cms/debug/interaction-v2.tsx`)
- Components documented but never imported in production code
- Experimental CSS files not referenced in the build

Before suggesting use of any component not already imported somewhere in active code, verify it's part of the current stable baseline and not an abandoned experiment.

**Why:** The repository contains remnants of experiments and debug harnesses. These should not be accidentally promoted to production use.

## Authoritative Active Paths

### Frontend (Next.js)
- **Pages:** `frontend/pages/` (Next.js routing)
- **Components:** `frontend/components/`
  - Design system: `frontend/components/setu/` (canonical for dashboard UI)
  - Marketing: `frontend/components/marketing/`
  - CMS: `frontend/components/cms/`
  - UI primitives: `frontend/components/ui/`
- **Styles:** `frontend/styles/globals.css` (main CSS entry point)
- **State Management:** `frontend/lib/`, `frontend/contexts/`

### Backend (FastAPI)
- **Application:** `backend/app/`
  - Main entry: `backend/app/main.py`
  - Models: `backend/app/models.py`
  - Route modules: `backup_routes.py`, `billing_routes.py`, `cms_routes.py`, etc.
- **Tests:** `backend/tests/` (pytest suite)
- **Database Migrations:** `alembic/versions/`
- **Services:** `backend/app/services/`, `backend/app/monitoring/`

### Configuration & Infrastructure
- **Root config:** `package.json`, `requirements.txt`, `alembic.ini`
- **Ops:** `ops/` (systemd configs, nginx, backups)
- **CI/CD:** `.github/workflows/`

## Code Organization Patterns

### Adding New Routes (Backend)
- Use specific route files (`*_routes.py`) for feature domains
- Include router in `backend/app/main.py`
- Avoid adding directly to the monolithic `routes.py` (it exists for legacy consolidation)

### Adding New Pages (Frontend)
- Create in `frontend/pages/` following Next.js conventions
- Use appropriate layout from `frontend/components/layout/`
- Follow design system patterns from `frontend/components/setu/`

### Testing
- Backend: Add to `backend/tests/` using pytest
- Frontend: Add to `frontend/__tests__/` or co-locate with components

## Cleanup Status

The repository is in **Phase 2: Hardening & Guardrails**.

- ✅ Phase 1 (Restore/Verify): Complete
- 🔄 Phase 2 (Hardening): In progress
  - ✅ Audit complete
  - ✅ Guardrails established (this document)
  - ⏳ Deferred: Component cleanup, branch cleanup, documentation archival

Some files are marked as deletion candidates but not yet removed. Treat them as "do not use" until they're formally removed.

## Questions or Conflicts?

If unclear whether code is authoritative or legacy:
1. Check `docs/repo-guardrails.md` for current cleanup status
2. Verify the file is imported/used in active code
3. Check git history: recent commits from stable branches are authoritative
4. When in doubt, ask the developer before suggesting code from unfamiliar paths

**Last Updated:** March 8, 2026

---
> Source: [shivamjpatel1/2025-NOV-AI-WEBSITE-MVP](https://github.com/shivamjpatel1/2025-NOV-AI-WEBSITE-MVP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-13 -->
