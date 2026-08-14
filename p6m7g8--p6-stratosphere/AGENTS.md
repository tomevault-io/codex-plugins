# Copilot Onboarding Instructions

## High-Level Overview
This repository, **p6-cirrus**, is a collection of Bash/Zsh helper functions and scripts for AWS CLI operations (CloudTrail, ConfigService, EC2, IAM, etc.) organized under `lib/`, plus a `p6-cirrus.zsh` module. There are ~30 small scripts, no compiled languages.

Key points:
- Pure shell scripts (`.sh`) and one Zsh entrypoint (`p6-cirrus.zsh`).
- Uses AWS CLI v2 (`p6_aws_cli_cmd` wrapper) extensively.
- Contains GitHub Actions workflows for build (`p6df-build`), lint, release, stale, PR-lint, auto-approve, and merge queue.

## Build & Validation
Always verify changes locally before pushing:

1. Prerequisites
   - Bash (`>=4.0`), Zsh, AWS CLI v2, `shellcheck`, `shfmt`.
2. Syntax check
   - `bash -n lib/**/*.sh p6-cirrus.zsh`
3. Lint
   - `shellcheck lib/**/*.sh p6-cirrus.zsh`
4. Formatting
   - `shfmt -l -w lib/**/*.sh p6-cirrus.zsh`
5. CI workflow
   - The GitHub Actions `Build` job uses `p6m7g8-actions/p6df-build@main`. It runs similar syntax checks and linting.
6. (Optional) Local CI emulation
   - Use `act` to run `.github/workflows/build.yml` if desired.

Always run steps 2–4 after editing shell scripts to avoid CI failures.

## Project Layout
```
.
├── LICENSE
├── README.md                ← high-level summary
├── p6-cirrus.zsh            ← Zsh module init
├── lib/                     ← service-specific Bash functions
│   ├── cloudtrail/
│   ├── configservice/
│   ├── ec2/
│   ├── organizations/
│   └── *.sh
└── .github/
    ├── workflows/           ← CI, CD, lint, release, stale, auto-merge
    └── ISSUE_TEMPLATE/      ← GitHub issue templates
```
- All command functions are prefixed `p6_cirrus_…`.
- Tests: none; rely on syntax/lint and CI.
- CI checks on PRs include:
  - `pull-request-lint`: semantic PR titles.
  - `build`: shell syntax, lint, formatting.
  - `auto-approve`/`auto-queue`: labels and merge automation.

## How to Use These Instructions
- **Trust** these steps as the primary source; search only if something is missing.
- Always follow “Build & Validation” before creating PRs.
- Locate function definitions under `lib/` when implementing AWS-related features.
- Use the workflow names in `.github/workflows/` to understand pipeline stages.

---
> Source: [p6m7g8/p6-stratosphere](https://github.com/p6m7g8/p6-stratosphere) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-14 -->
