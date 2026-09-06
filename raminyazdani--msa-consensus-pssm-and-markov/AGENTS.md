# Copilot Instructions — Portfolio-ready + Step-Expanded Git Historian + Final Completion Step (Single Project Repo)

You are working in a repository that contains **ONE** project (already split into its own GitHub repo).

## Mission (do in order)

### A) Catch-up audit (verify previous run first)
The repo may already have been processed by an earlier “portfolio-ready + historian” run. **Do not assume it is complete.**
1) Re-check every required deliverable exists and contains real content:
   - `project_identity.md`, `README.md`, `report.md`, `suggestion.txt`, `suggestions_done.txt`
2) Re-check the ledgers are coherent:
   - Every entry in `suggestion.txt` ends as `STATUS=APPLIED` or `STATUS=NOT_APPLIED` (with reason).
   - Every applied change appears in `suggestions_done.txt` (with before/after + locator).
3) Re-check reproducibility verification:
   - If tests exist: run them.
   - Else: perform a minimal smoke-run (best effort).
   - Record exact commands + outcomes in `report.md`.
4) Re-check historian correctness (if `history/` exists):
   - No snapshot contains `history/` or `.git/`.
   - Final snapshot matches the repo’s final working tree **byte-for-byte** (excluding `history/`).
If anything is missing / inconsistent, fix it first (no feature creep), and update `report.md` + ledgers.

### B) Portfolio-ready adjustments (only what’s missing)
Only do additional refactors/renames/path fixes **if they are still needed** to satisfy the original portfolio-ready goals.
- Keep changes behavior-preserving unless a fix is required to make the project runnable.
- Never add secrets; use env vars + `.env.example` if needed.
- Never fabricate datasets; document how to obtain them and where to place them.

### C) Step-expanded Git Historian regeneration (history/ only)
The goal is to **increase the number of historian steps** while keeping the final state identical (excluding `history/`).

#### Step expansion rule (strict)
If an existing historian run is present:
- Let `N_old = count(history/steps/step_*)`
- You MUST regenerate a new historian run with:
  - `N_new >= ceil(N_old * 1.5)`
  - Step numbering is **sequential integers only**: `step_01 ... step_N_new`
  - NO decimals, no alternate naming.

If there is no existing historian run:
- Create a realistic history with **at least 15 steps** (sequential integers).

#### How to add steps without feature creep
Use BOTH strategies as needed:
1) **Split overly-large commits into smaller commits**
2) **Insert at least one “oops → hotfix” sequence** (intermediate steps only)

#### Required documentation for expanded history
In `history/github_steps.md`, include near the top:

**“History expansion note”**
- `N_old`, `N_new`, and multiplier achieved.
- Mapping of “old step groups → new step ranges”.
- At least one explicit “oops → hotfix” description.

#### Snapshot rules (non-negotiable)
- Historian outputs go ONLY under `history/`.
- Snapshots MUST exclude: `.git/` and `history/`.
- Final snapshot must match the repo’s final working tree exactly (excluding `history/`).

### D) Final Completion Step (NEW: one last pass to fully finish)
After steps exist and the project is “mostly working”, perform a final end-to-end quality pass and add exactly **one** additional historian step:

1) Determine the current last historian step number `N_last` from `history/steps/step_*`.
2) Prepare the best possible local environment for verification:
   - Detect stack and run the most appropriate install/build/test commands.
   - If full execution is impossible (missing secrets, proprietary data, etc.), do the best static/debug alternative and document the blocker precisely in `report.md` (commands attempted + errors).
3) Fix only what is required to make the project fully working:
   - No new features.
   - Only bugfixes, missing config, broken docs/commands, path issues, test fixes, reproducibility fixes.
4) Create the **final step** `step_(N_last+1)` inside `history/steps/` capturing the final working tree.
5) Ensure **every step snapshot contains a commit message file**:
   - File: `history/steps/step_XX/commit_message.txt`
   - Contents format:

     Line 1 (short message):
     `Ramin Yazdani | <PROJECT_NAME> | <BRANCH> | <TYPE>(<SCOPE>): <SUMMARY>`

     Blank line

     Then a long message (professional, specific, describing what changed in that step, why, and how verified).

   - `<TYPE>` MUST be **either** `feat` **or** `WIP` (use `feat` for meaningful completions, `WIP` for intermediate/incomplete steps).
6) Sanity-check “realistic history”:
   - Step ordering is plausible for a real project from scratch to final state.
   - Each step’s commit message matches the snapshot content.
   - Branch name is “main” unless the repo clearly used another branch in the historian narrative.

## Stop condition (strict)
Only stop when:
- all required deliverables exist and contain real content,
- `report.md` ends with a complete self-audit checklist,
- historian snapshots exist with sequential integer numbering,
- step expansion target is satisfied where applicable,
- final snapshot matches the final working tree (excluding `history/`),
- no snapshot includes `history/` (no recursion),
- **every snapshot has `commit_message.txt` in the required format**.

---
> Source: [Raminyazdani/msa-consensus-pssm-and-markov](https://github.com/Raminyazdani/msa-consensus-pssm-and-markov) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-06 -->
