# Copilot Instructions

## Project: Compute Desk

This is a Copilot CLI extension pack (`@adtork/compute-desk`) providing 20 specialist
agents for Azure IaaS virtual machine tasks — SKU sizing, cost, performance, availability,
disaster recovery, backup, scale sets, disks, NVA VMs, security, identity, images,
patching, monitoring, migration, GPU/HPC, licensing, troubleshooting, OS guest ops, and
report building (polished Markdown/HTML/PDF/DOCX/XLSX deliverables).

## Build & Run

```
# No build step — pure ES modules, zero npm dependencies.
node bin/cli.mjs init             # Install extension to ~/.copilot/extensions/
node bin/cli.mjs init --project   # Install extension to .github/extensions/ in current repo
node bin/cli.mjs update           # Re-install over existing user/project installs
node bin/cli.mjs status           # Check installation (shows installed version)
node bin/cli.mjs uninstall        # Remove
node bin/cli.mjs --version        # Print version
```

## Architecture

- `bin/cli.mjs` — CLI installer (init, init --project, update, status, uninstall, --version).
  Writes `.install-meta.json` (version + install type) into the install directory.
- `extensions/compute-desk/registry.mjs` — **`REGISTRY` (single source of truth)**. A
  dependency-free ES module describing every specialist: `dir`, `domain`, `name`, `summary`,
  `icon`, `trigger` (routing regex), `guidance`, and `skills` ({name: description}).
- `extensions/compute-desk/extension.mjs` — Main SDK extension. Imports `REGISTRY` and
  registers exactly **5 parameterized tools**. Handles routing hints, presence announcement,
  file caching, opt-in validation, and a throttled update check.
- `extensions/compute-desk/specialists/<dir>/` — Each specialist has `agents/<dir>.md`
  (role definition) and `skills/<skill>/SKILL.md` (deep skill guidance).
- `extensions/compute-desk/renderers/` — Python report renderers used by the
  `report-builder` (`cd_doc`) specialist: `make_html.py`, `make_pdf.py`, `make_docx.py`,
  `make_xlsx.py`. Tools never invoke these; the agent runs them per the report-builder
  skills, writing to `compute-desk/<specialist>/reports/`.

## Key Conventions

- **5 tools only — hard 128-tool API limit.** Copilot CLI rejects extensions that register
  more than 128 tools. Never register per-specialist or per-skill tools. The 5 tools are:
  `cd_capabilities`, `cd_route`, `cd_role({ specialist })`,
  `cd_orchestrate({ specialist })`, `cd_skill({ specialist, skill })`.
- **Single source of truth:** the `REGISTRY` object in `registry.mjs`. Routing, the
  capabilities table, orchestration prompts, and skill loading all derive from it.
  `registry.mjs` imports nothing so it can load outside the CLI host.
- **Specialist ids are `cd_`-prefixed (internal only):** `cd_sku`, `cd_cost`, `cd_perf`, `cd_avail`, `cd_dr`,
  `cd_bkp`, `cd_vmss`, `cd_disk`, `cd_nva`, `cd_sec`, `cd_iam`, `cd_img`, `cd_patch`,
  `cd_mon`, `cd_mig`, `cd_gpu`, `cd_lic`, `cd_ts`, `cd_os`, `cd_doc`. These ids are internal keys;
  the `specialist` tool argument is the **friendly name** (e.g. "SKU & Sizing"). The
  `SPECIALIST_PARAM` enum is `REGISTRY[p].name`, and `resolveSpecialist()` accepts the friendly
  name, domain, dir, short id, or `cd_*` id. Surface friendly names (not `cd_*` ids) to the user.
- **Routing:** Regex-based keyword detection in `onUserPromptSubmitted`; the extension
  announces itself once via `onSessionStart` and injects per-specialist hints (throttled).
- **All tools use `skipPermission: true`** — they only read bundled Markdown, never mutate.
- **No external npm dependencies** — only `@github/copilot-sdk/extension` and `node:*`.
- **Guardrail:** Every specialist role + skill file ends with a validation-first disclaimer
  ("Validation-first: verify every Azure fact against the Microsoft Learn MCP server … Analysis
  only — verify against Microsoft documentation before applying.").
- **Validation-first MCP policy (centralized).** The extension treats the Microsoft Learn MCP
  server as the primary source of truth. The policy lives once in `extension.mjs` as
  `MCP_VALIDATION_DIRECTIVE` (+ the `MCP_FALLBACK_BANNER` constant) and is surfaced on every
  specialist-loading path: `cd_role`/`cd_skill` outputs are wrapped by the `withPolicy()` helper,
  and the directive is included in `cd_orchestrate` (`buildOrchestrator`), the
  `onUserPromptSubmitted` routing hints, and `PRESENCE_NOTE`. Update the policy in that one
  constant — do NOT re-author it per specialist.
- **Adding a specialist or skill:** edit `registry.mjs` (add the entry/skill) and add the
  matching `agents/<dir>.md` / `skills/<skill>/SKILL.md` files. No other code changes are
  needed — the extension derives everything from the registry.
- **Opt-in checks:** set `COMPUTE_DESK_VALIDATE=1` to validate every role/skill file exists
  on load; set `COMPUTE_DESK_NO_UPDATE_CHECK=1` to disable the update check.
- **Keep the version and CHANGELOG in sync (required).** `package.json` `version` is the
  single source of truth (read by `bin/cli.mjs`, written to `.install-meta.json`, reported by
  `status`/`--version`, and compared by the runtime update check). There is no tooling that
  syncs them, so every release MUST, in the same change: (1) bump `package.json` `version`
  (semver — additive features = minor), and (2) move the `## [Unreleased]` CHANGELOG entries
  under a new `## [x.y.z] - YYYY-MM-DD` heading (leaving a fresh empty `## [Unreleased]` on top).
  Never bump one without the other.

---
> Source: [adtork/compute-desk](https://github.com/adtork/compute-desk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-26 -->
