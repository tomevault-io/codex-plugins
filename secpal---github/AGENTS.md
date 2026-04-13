<!--
SPDX-FileCopyrightText: 2026 SecPal
SPDX-License-Identifier: AGPL-3.0-or-later
-->

# SecPal Copilot Instructions

These instructions are authoritative when working inside the `.github` repository itself.
Other repositories must keep their own self-contained runtime instructions.

## Always-On Rules

- Run `git status --short --branch` before any write action. New work must start
  from a clean, up-to-date local `main`: switch to `main`, pull with
  fast-forward only, verify a clean state, then create the dedicated topic
  branch. Never start implementation on local `main`, and stop if a dirty
  non-`main` branch contains unrelated work.
- TDD is mandatory for behavior, automation, or executable policy changes. Add or update the smallest relevant failing test or validation first, then implement, then refactor with checks green.
- Quality first. Do not trade correctness, review depth, validation depth, or issue tracking for speed.
- Keep one topic per change. 1 topic = 1 PR = 1 branch. Do not mix unrelated fixes, features, refactors, docs, or governance cleanup.
- Never use bypasses such as `--no-verify` or force-push.
- Update `CHANGELOG.md` for real fixes, features, and breaking changes in the same change set.
- Create a GitHub issue immediately for every real out-of-scope bug, technical debt, missing test, documentation gap, warning, audit finding, or deprecation you cannot fix now. Do not leave untracked `TODO`, `FIXME`, or "follow-up" work.
- Use EPIC plus sub-issues before implementation whenever work will span more than one PR; if in doubt, choose EPIC plus sub-issues.
- Keep GitHub-facing communication in English and reference files and lines instead of pasting large code blocks.
- Treat warnings, audit findings, and deprecations as actionable. Fix them in scope or track them immediately.
- Keep `SPDX-FileCopyrightText` years current in edited files or companion `.license` sidecars.
- Never reply to Copilot review comments with GitHub comment tools. Fix the code, push, and resolve threads
  using the approved non-comment workflow (`docs/copilot-review-automation.md` or `scripts/copilot-review-tool.sh`).
- After every merge, immediately return the local repo to a ready state:
  switch to `main`, pull with fast-forward only, delete the merged topic
  branch, prune remotes, refresh dependencies where applicable as the
  smallest relevant readiness step for this repository, and confirm the
  working tree is clean.

## Design Principles

- DRY: eliminate duplicated logic and repeated policy text; extract shared guidance before it drifts.
- KISS: prefer the simplest solution that satisfies the current requirement and remains obvious in six months.
- YAGNI: implement only what the current issue, PR, or acceptance criteria require; avoid speculative abstractions.
- SOLID: keep responsibilities narrow, interfaces small, and extension points explicit.
- Fail fast: stop on the first failed check, fix the problem, and do not accumulate known breakage.

## Required Validation

Before any commit, PR, or merge, announce the checklist you are executing and stop on the first failed item.
At minimum verify:

- the active branch and PR scope still address exactly one topic
- TDD happened where executable behavior or validation changed
- the smallest relevant validation for the touched area passed, and `./scripts/preflight.sh` ran for substantial governance or workflow changes
- out-of-scope findings were turned into GitHub issues immediately
- `CHANGELOG.md` was updated for real changes
- commits are GPG-signed
- REUSE compliance was checked when changed files require it
- when a fix alters observable behavior, validation rules, or automation logic, the corresponding tests or validations were identified and updated in the same commit
- the local 4-pass review was completed, including DRY, KISS, YAGNI, SOLID, quality-first, and issue-management checks
- no bypass was used

## Issue And PR Discipline

- Every real out-of-scope finding becomes a GitHub issue immediately; no untracked follow-up work is allowed.
- Complex work uses EPIC plus sub-issues before implementation. PRs should close sub-issues, not the epic, until the final linked step.
- When local review finds zero issues, commit and push the finished branch before opening any PR.
- The first PR state must be draft. Do not open a normal PR first.
- Mark a draft PR ready only after the final self-review in the PR view still finds zero issues.
- When creating or editing PRs programmatically, write multi-line body content to a file and use `--body-file` to prevent shell escaping issues.

## Local Review Standard

Run these four passes before creating a PR:

1. Comprehensive review: correctness, tests, docs, no stray TODOs.
2. Deep-dive review: domain policy, licensing, security-sensitive patterns.
3. Best-practices review: hidden files, governance docs, package metadata, workflow hygiene.
4. Security review: explicit permissions, secret handling, ignore rules, automation safety.

When all passes are clean, commit and push before creating the draft PR.

## Domain Policy

Use only these domains and identifiers:

- `secpal.app` for the public homepage and real email addresses
- `apk.secpal.app` for the canonical Android artifact and download host
- `api.secpal.dev` for the live API host
- `app.secpal.dev` for the live PWA/frontend host
- `secpal.dev` for dev, staging, testing, and examples
- `app.secpal` only as the Android application identifier

Treat `api.secpal.app` as a deprecated web host.

## Android Distribution Policy

- Ship one Android application package: `app.secpal`.
- Keep one signing identity and one version line across Play Store, GitHub Releases, Obtainium, direct APK downloads, and Device Owner provisioning.
- Use `apk.secpal.app` as the technical APK/checksum/release-metadata host.
- Use `secpal.app/android` as the human-facing Android landing surface.
- Provisioning QR codes must be generated inside SecPal for authorized users only.
- Provisioning QR payloads must stay minimal and use short-lived bootstrap tokens instead of public static policy blobs or long-lived secrets.

## Repository Conventions

- This repository is not versioned; keep its changelog chronological.
- Hidden files and automation files are first-class source artifacts.
- For GitHub workflows, set explicit permissions, set `timeout-minutes` on every job, pin external actions, and never expose secrets in logs.
- Workflow templates for other repositories live in `workflow-templates/`.
  Reusable workflow definitions live in `.github/workflows/reusable-*.yml` and are invoked via `uses:`.
- Keep changes repo-local, minimal, and aligned with the existing governance and automation patterns.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SecPal)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/SecPal)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
