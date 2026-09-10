<!-- Copyright (c) Microsoft Corporation. -->
<!-- Licensed under the MIT License. -->

# Copilot / agent instructions — intune-my-macs

Read **[../AGENTS.md](../AGENTS.md)** first — it is the canonical entry point and
describes the repository layout, the manifest model, the build/run/validate
loop, and where the standards live.

## Working rules

- **Validate before pushing:** run `pwsh ./scripts/verify.ps1`. It checks that
  every JSON, XML, and `.mobileconfig` artifact parses and that PowerShell
  scripts are syntactically valid.
- **Follow the conventions:** [../docs/conventions.md](../docs/conventions.md),
  [../standards/policy-naming-standard.prd](../standards/policy-naming-standard.prd),
  and [../standards/manifest-standard.prd](../standards/manifest-standard.prd).
  Any new artifact needs a sibling `.xml` manifest with a unique `<ReferenceId>`,
  placed under the relevant platform folder (e.g. `macOS/`).
- **Generated docs:** never hand-edit `INTUNE-MY-MACS-DOCUMENTATION.md`;
  regenerate it with `python3 tools/Generate-ConfigurationDocumentation.py` when
  artifacts change.
- **Copyright headers:** add the Microsoft header to every new first-party file;
  never remove a pre-existing third-party copyright notice.

## Pull requests

- Keep PRs **reasonably sized** and split large work into logical units (one
  concern per commit/PR).
- Update `CHANGELOG.md` (newest first) and link every file path you mention.
- Never use `--no-verify`; never force-push or `git reset --hard` published
  commits.
- The end-to-end release flow is in
  [prompts/ship.prompt.md](prompts/ship.prompt.md).

---
> Source: [microsoft/intune-my-macs](https://github.com/microsoft/intune-my-macs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-10 -->
