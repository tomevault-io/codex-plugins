# Copilot Instructions

## Project: Network Desk

This is a Copilot CLI extension pack (`@dmauser/network-desk`) providing 20 specialist agents for cloud networking tasks across Azure, AWS, and GCP, plus 14 firewall vendor platforms.

## Build & Run

```
# No build step — pure ES modules
node bin/cli.mjs init          # Install extension to ~/.copilot/extensions/
node bin/cli.mjs init --project # Install extension to .github/extensions/ in current repo
node bin/cli.mjs init --plugin  # Install as a native Copilot CLI plugin (copilot plugin)
node bin/cli.mjs plugin build   # (Re)generate the plugin bundle from the registry
node bin/cli.mjs status        # Check installation
node bin/cli.mjs uninstall     # Remove
```

## Architecture

- `bin/cli.mjs` — CLI installer (init, init --plugin, plugin build, update, status, uninstall)
- `extensions/network-desk/registry.mjs` — **`REGISTRY` (single source of truth)**, exported as a dependency-free ES module (no SDK/`node:*` imports) so it can be shared by the extension runtime and the plugin generator
- `extensions/network-desk/extension.mjs` — Main SDK extension: imports `REGISTRY` from `./registry.mjs`, defines parameterized tool handlers + session hooks via `@github/copilot-sdk/extension`
- `extensions/network-desk/scripts/build-plugin.mjs` — Generator that emits the native Copilot CLI plugin (`plugins/network-desk/**` + repo-root `.github/plugin/marketplace.json`) from `REGISTRY` + the `specialists/**` files. Builds to a staging dir, validates, then swaps in. Run via `node bin/cli.mjs plugin build`.
- `plugins/network-desk/` — **Generated, do not hand-edit.** One coordinator agent (`agents/network-desk.md`) + 20 namespaced specialist skills (`skills/network-desk-<prefix>/`), each bundling its deep sub-skill docs under `reference/`. Regenerate after editing `REGISTRY`.
- `extensions/network-desk/specialists/<name>/` — Each specialist has:
  - `agents/<name>.md` — Agent role definition (persona, workflow, guardrails)
  - `skills/<skill-name>/SKILL.md` — Deep domain expertise for each skill
- All specialists use **Pattern B** (separate files), not Pattern A (single SKILLS.md)

## Key Conventions

- **5 tools only — hard 128-tool API limit.** The CLI/model rejects sessions exposing >128 tools (request fails with "transient API error. Retrying..."). With 20 specialists × (role + orchestrate + ~6 skills) this would be ~162 tools, so tools are **parameterized**, not registered per-specialist. NEVER register one tool per specialist/skill.
- **The 5 tools:** `cn_capabilities`, `cn_route`, `cn_role({ specialist })`, `cn_orchestrate({ specialist })`, `cn_skill({ specialist, skill })`.
- **Single source of truth:** the `REGISTRY` object lives in `extensions/network-desk/registry.mjs` (prefix → `{ dir, domain, name, summary, icon, trigger, guidance, skills }`) and generates routing, capabilities, orchestration prompts, skill loading, **and the generated plugin bundle**. Add specialists/skills by editing `registry.mjs` only, then run `node bin/cli.mjs plugin build` to regenerate `plugins/network-desk/`.
- **Specialist ids are `cn_`-prefixed:** `cn_vnet`, `cn_fw`, `cn_lb`, `cn_dns`, `cn_pl`, `cn_hyb`, `cn_nsec`, `cn_ntsh`, `cn_vwan`, `cn_nmon`, `cn_mcn`, `cn_price`, `cn_iac`, `cn_cnet`, `cn_cdn`, `cn_nauto`, `cn_sase`, `cn_ncap`, `cn_ipv6`. The `REGISTRY` is keyed by the bare prefix internally; `pub(prefix)` produces the `cn_` id, and `resolveSpecialist()` accepts the `cn_` id, the bare alias, or the directory name.
- **Legacy names are references only:** strings like `cn_vnet_skill_address_planner` / `vnet_skill_address_planner` are NOT callable tools; they're tolerated as the `skill` argument to `cn_skill` (normalized to kebab-case).
- **Routing:** Regex-based keyword detection in `onUserPromptSubmitted` injects MUST-language guidance pointing at the parameterized tools (throttled once per specialist per session); an `onSessionStart` presence note announces the extension from turn 1. The hook forbids reading `specialists/**` files directly and requires loading content via `cn_role`/`cn_orchestrate`/`cn_skill`.
- **All tools use `skipPermission: true`** — they deliver read-only content (markdown), not actions
- **No external dependencies** — only `@github/copilot-sdk/extension` and `node:*` builtins
- **Guardrail:** Every specialist output ends with "Analysis only — verify against vendor documentation before applying."
- **Validation-first MCP policy (centralized, per-cloud).** The extension treats each cloud's official documentation MCP server as the primary source of truth: **Azure** → Microsoft Learn MCP (`microsoft-learn`, hosted HTTP), **AWS** → AWS Documentation MCP (`aws-docs`, local stdio via `uvx awslabs.aws-documentation-mcp-server@latest`), **GCP** → a configurable `gcp-docs` placeholder. The directive lives in `registry.mjs` as `MCP_PROVIDERS` (per-cloud server/transport/addCommand/tools) → `MCP_VALIDATION_DIRECTIVE` (+ `mcpFallbackBanner()`/`MCP_FALLBACK_BANNER` and the compact `MCP_VALIDATION_NOTE`) and is surfaced on every `cn_role`/`cn_skill` load (via `withPolicy()`), in `buildOrchestrator`, in the presence note, and in the prompt hook whenever a specialist trigger matches — and is mirrored into the generated plugin (coordinator agent + each specialist `SKILL.md`) by `build-plugin.mjs`. Each cloud's facts must be docs-MCP-validated ("the docs MCP wins"; cite URLs) or fall back to a ⚠️ unverified banner with that cloud's `copilot mcp add` command. Each docs MCP is its own cloud only — firewall-vendor facts keep the vendor-doc guardrail. Network Desk never edits MCP config (per-user CLI setting). To add/change a cloud's server, edit `MCP_PROVIDERS` then rebuild the plugin.
- **14 firewall vendors** in firewall-engineer: Azure FW, AWS NFW, GCP FW, PAN-OS, FortiGate, Check Point, Cisco ASA/FTD, Juniper SRX, Zscaler, Sophos XG, OPNsense, pfSense, VyOS, iptables/nftables

---
> Source: [dmauser/network-desk](https://github.com/dmauser/network-desk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-07 -->
