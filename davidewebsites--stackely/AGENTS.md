# Stackely Development Rules for AI Agents

## Core Principles

- **Minimal changes only**: Each task should modify the smallest number of files possible
- **No environment files**: Do NOT touch `.env`, `.env.local`, `.env.development`
- **No unnecessary refactoring**: Only fix what's explicitly requested, don't "improve" unrelated code
- **Preserve existing functionality**: Never remove code unless explicitly told to do so
- **Incremental commits**: Each change should be a logical, reviewable unit

---

## Project Structure

```
Stackely/
├── app/frontend/          # Vite + React + TypeScript
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── lib/
│   │   ├── api/
│   │   ├── hooks/
│   │   └── contexts/
│   ├── package.json
│   └── pnpm-lock.yaml
├── app/backend/           # FastAPI
│   ├── app/
│   └── requirements.txt
├── automation/            # Automation & scripts
│   ├── run_cycle.ps1
│   ├── task.json
│   ├── changed_files.txt  # Generated
│   └── last_diff.patch    # Generated
└── CLAUDE.md             # This file
```

---

## Key File Locations

| File | Purpose | Edit? |
|------|---------|-------|
| `app/frontend/src/pages/Results.tsx` | Search results page | ✓ Yes |
| `app/frontend/src/lib/api.ts` | API layer, functions | ✓ Yes |
| `app/frontend/src/api/searchTools` | Search implementation | ✓ Yes |
| `app/frontend/package.json` | Dependencies, scripts | ⚠️ Only if adding dep |
| `.env` | Secret configuration | ✗ NO |
| `.env.local` | Local overrides | ✗ NO |

---

## Before Starting a Task

1. **Read the task schema** in `automation/task.json`
2. **Check constraints**: Explicitly list what you must NOT do
3. **Verify scope**: Stay within the listed files/folders
4. **Check acceptance criteria**: You must satisfy all of them
5. **Run validation**: Use `run_cycle.ps1` to verify your changes

---

## Expected Output Format

Every task completion must include:

```
## Summary
[1-2 sentence description of what was done]

## Files Changed
- app/frontend/src/pages/Results.tsx (lines X-Y)
- app/frontend/src/lib/api.ts (new function)

## Validation
- Build: PASSED / FAILED
- Lint: PASSED ([X warnings])
- Git diff: [X insertions, Y deletions]

## Risks
- [List any potential issues or edge cases]
```

---

## Frontend Build & Test Commands

```bash
cd app/frontend

# Build
pnpm build

# Lint
pnpm lint

# Dev server
pnpm dev

# Preview production build
pnpm preview
```

---

## Automation Script Usage

From root, run in PowerShell:

```powershell
.\automation\run_cycle.ps1
```

This will:
1. Show current git status
2. Run `pnpm build` in frontend
3. Run `pnpm lint` in frontend
4. Generate diff patch
5. List changed files

**Outputs saved to:**
- `automation/changed_files.txt`
- `automation/last_diff.patch`

---

## Common Constraints

- **No styling changes** unless specified
- **No component refactoring** unless specified
- **No dependency upgrades** unless specified
- **No test file modifications** unless specified
- **No breaking API changes** unless specified
- **Preserve backward compatibility** in public functions

---

## Helpful Commands During Development

```bash
# Check git status
git status

# See what changed since last commit
git diff

# See specific file changes
git diff app/frontend/src/pages/Results.tsx

# Check TypeScript errors
cd app/frontend && pnpm lint

# Generate new .patch file
git diff > automation/last_diff.patch

# List files not in .gitignore
git status --porcelain
```

---

## If Something Goes Wrong

1. **Build fails?** → Check `npm install` / `pnpm install` in frontend
2. **Lint fails?** → Review errors, fix in the file, re-run `pnpm lint`
3. **Can't find file?** → Check paths are relative to project root
4. **Environment error?** → Do NOT modify .env, check if var is used correctly

---

Version: 1.0  
Last Updated: 2026-03-17

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Davidewebsites)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Davidewebsites)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
