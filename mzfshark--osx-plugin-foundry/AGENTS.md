# Copilot Instructions — Aragon OSx Plugin (Foundry)

Foundry template for building, testing, and deploying Aragon OSx plugins with comprehensive tooling.

## Big Picture

- **Plugin variants**: UUPS upgradeable (`MyUpgradeablePlugin.sol`), cloneable (`MyCloneablePlugin.sol`), and static (`MyStaticPlugin.sol`) implementations available.
- **Aragon OSx integration**: Plugins integrate with DAO framework via `PluginSetup` contracts that define install/uninstall/update lifecycles.
- **Custom plugins**: Harmony-specific voting plugins (HIP, Delegation, Validator Opt-In) under `src/harmony/`; use as reference for production implementations.
- **Deployment flows**: Three strategies—simple plugin repo, full DAO with plugins, and trustless factory-based—choose via `DEPLOYMENT_SCRIPT` in `Makefile`.
- **Dependencies**: Aragon OSx libraries (`@aragon/osx`, `@aragon/osx-commons`), OpenZeppelin contracts, and Forge standard library.

## Planning & Issue Tracking Workflow

**CRITICAL: After completing planning and BEFORE starting implementation:**

1. **Generate Plan Document**: Create `PLAN.md` at repository root containing:

   - [ ] Clear task breakdown with checkboxes
   - [ ] Implementation steps and guidelines
   - [ ] Dependencies and integration points
   - [ ] Expected outcomes and acceptance criteria

2. **Sync with GitHub Project**: Using GitHub CLI (`gh` - already authenticated as mzfshark):

   ```bash
   # Create issue from PLAN.md
   gh issue create --title "[Plan] <descriptive-title>" --body-file PLAN.md --project "https://github.com/users/mzfshark/projects/5"
   ```

3. **Update Plan Progress**: As tasks complete, update checkboxes in `PLAN.md` and sync with issue:
   ```bash
   # Update the issue body with current PLAN.md
   gh issue edit <issue-number> --body-file PLAN.md
   ```

**IMPORTANT**: Never run `git commit` or `git push` automatically. Always ask the user before any git operations.

**Never start implementation without a documented plan in `PLAN.md` and corresponding GitHub issue.**

## Tool Restrictions

**FORBIDDEN: Do NOT use `codacy_get_pattern` tool** — This tool is incompatible with WSL environments and will fail. Use alternative Codacy tools for code quality analysis.

## Language Standards

**MANDATORY: All public-facing content MUST be in English:**

- **Code comments**: All comments in source code must be written in English
- **Logs and console output**: All log messages, debug output, and error messages must be in English
- **GitHub Issues**: All issue titles, descriptions, and comments must be in English
- **Commit messages**: All git commit messages must be in English following conventional commits format
- **Documentation**: All README files, inline docs, and API documentation must be in English
- **Variable/function names**: Use English for all identifiers in code

**Examples:**
```bash
# ✅ CORRECT
git commit -m "feat: implement Band Oracle integration for HIP plugin"

# ❌ INCORRECT
git commit -m "implementa integração com Band Oracle para plugin HIP"
```

**Note**: This standard ensures international collaboration and maintainability. Internal planning documents (like `PLAN.md` for local work) may use Portuguese if needed, but all published content must be English.

## Key Paths

- Plugin implementations: `src/*.sol` (template variants), `src/harmony/*.sol` (production Harmony plugins).
- Setup contracts: `src/setup/MyPluginSetup.sol` (defines installation logic and permissions), `src/harmony/*Setup.sol`.
- Deploy scripts: `script/Deploy*.s.sol`—choose via `DEPLOYMENT_SCRIPT` in `Makefile`.
- Test files: `test/*.t.sol` (Solidity tests), `test/*.t.yaml` (YAML test definitions converted via Bulloak).
- Test builders: `test/builders/SimpleBuilder.sol` (local), `test/builders/ForkBuilder.sol` (fork testing with OSx factories).
- Config: `foundry.toml`, `.env` (copy from `.env.example`), `Makefile`.
- Artifacts: `artifacts/` (deployment outputs), `deployed_contracts_harmony.json` (Harmony addresses).

## Install, Build, Test

- **Setup**: `make init` (checks dependencies, runs first build). Copy `.env.example` to `.env` and configure.
- **Build**: `forge build` or `make` (via Makefile).
- **Test**: `make test` (unit), `make test-fork` (fork tests using `RPC_URL`), `make test-coverage` (HTML report).

## Terminal Timing (tests/build/type-check)

After running `test`, `type-check`, or `build` commands, wait 120 seconds before attempting to read terminal output.
- **Test scaffolding**: `make sync-tests` (scaffolds/syncs `.t.yaml` → `.t.sol` via Bulloak), `make check-tests` (checks sync status).
- **Test tree**: `make test-tree` (generates `TESTS.md` from `.t.yaml` files).

## Deployment Workflow

1. **Pre-deploy simulation**: `make predeploy` (dry-run).
2. **Deploy**: `make deploy` (deploys, verifies, writes to `artifacts/`).
3. **Retry/resume**: `make resume` (retry failed txs).
4. **Verification**: `make verify-etherscan`, `make verify-blockscout`, `make verify-sourcify`.
5. **Update registry**: Use `script/update-deployed-contracts.sh <contract> <address> <tx_hash>` to update `deployed_contracts_harmony.json`.

Set `DEPLOYMENT_SCRIPT` in `Makefile` to choose deployment strategy:

- `DeploySimple` → simple plugin repo (trusted).
- `DeployDaoWithPlugins` → full DAO with plugins installed (trusted).
- `DeployViaFactory` → trustless factory-based.

## Plugin Setup Conventions

- **Installation parameters**: Encode in `prepareInstallation()`, decode with helper (e.g., `decodeInstallationParams()`).
- **Permissions**: Request via `PreparedSetupData.permissions[]`—define `PermissionLib.MultiTargetPermission` entries for `GRANT`, `REVOKE`, `GRANT_WITH_CONDITION`.
- **Upgrades (UUPS only)**: Use `PluginUpgradeableSetup` and implement `prepareUpdate()`.
- **Helpers**: Include `PluginSetupProcessorHelpers.sol` or similar for test/deploy address resolution.

## Conventions & Patterns

- **Contract variants**: Toggle variant in `MyPluginSetup.sol` constructor and `prepareInstallation()` (uncomment desired variant).
- **Test builders**: Use `SimpleBuilder` for local tests, `ForkBuilder` for integration tests with real OSx factories.
- **YAML test definitions**: Define test structure in `.t.yaml`, scaffold Solidity via `make sync-tests`, implement logic in `.t.sol`.
- **LLM helpers**: `make test-tree-prompt src=<file>` generates prompt for AI to write test YAML; `make test-prompt src=<file>` for test implementation.
- **Multi-verifier support**: Supports Etherscan, BlockScout, Sourcify, zkSync, Routescan via `VERIFIER` env var.
- **Chain-specific params**: `Makefile` adjusts Forge params based on `CHAIN_ID` (e.g., zkSync uses `--zksync` build flag).

## Harmony Plugin Specifics

- **HIP Plugin**: Band Oracle integration, allowlist control via `HIPPluginAllowlist` UUPS proxy.
- **Delegation Plugin**: Configurable `validatorAddress` storage, validator opt-in registry (`HarmonyValidatorOptInRegistry`).
- **Deploy**: Use `script/DeployHarmonyVotingRepos.s.sol`; configure `.env` with `ORACLE_ADDRESS`, `MANAGEMENT_DAO_ADDRESS`, etc.
- **Allowlist management**: After deploy, use `HIPPluginAllowlist.allowDAO(daoAddress)` to permit DAOs.
- **Addresses**: Track in `deployed_contracts_harmony.json`; sync with backend's `MANAGEMENT_DAO_PROXY_ADDRESS`.

## Cross-Repo Integration

- **Aragon OSx (contracts)**: Plugin setups depend on OSx core (`@aragon/osx`) and commons (`@aragon/osx-commons`); align versions via `forge install` and commit submodule refs.
- **Backend**: Deployed addresses must match backend's config (e.g., `MANAGEMENT_DAO_ADDRESS` in backend env).
- **App**: Frontend consumes plugin ABIs/addresses; update app's network definitions after deployment.

## Example Tasks (Do It This Way)

- **Add a plugin**: Implement logic in `src/YourPlugin.sol`, create `src/setup/YourPluginSetup.sol`, define permissions, add tests in `test/YourPlugin.t.yaml`, scaffold via `make sync-tests`, implement tests.
- **Add a network**: Update `.env` with `NETWORK_NAME`, `CHAIN_ID`, `RPC_URL`, adjust `Makefile` if chain needs custom Forge flags, deploy, update `deployed_contracts_<network>.json`.
- **Update plugin version**: Create new setup contract (e.g., `YourPluginSetupV2.sol`), implement `prepareUpdate()` in upgradeable setup, publish new version to plugin repo.
- **Debug fork tests**: Use `make test-fork` with `VERBOSITY=-vvvv`, check `ForkBuilder.sol` for factory addresses and DAO setup logic.
- **Verify contracts**: After deploy, run `make verify-etherscan` (or other verifier), or use `script/verify-contracts.sh` for batch verification.

## Additional Notes

- **Bulloak**: Required for test scaffolding; install via `cargo install bulloak`.
- **Deno**: Required for YAML → tree conversion; install via `curl -fsSL https://deno.land/install.sh | sh`.
- **Docker**: Recommended for hermetic deployments; see `Dockerfile` (not visible in this workspace but may exist).
- **Storage layout**: `make storage-info` shows storage layout for upgradeable contracts (critical for UUPS upgrades).
- **Refund**: `make refund` refunds remaining balance from deployment account (useful for testnet cleanup).
- **LLM workflows**: Use `llm.mk` recipes for AI-assisted test generation; see `make test-tree-prompt` and `make test-prompt`.

## wsl Notes
- do not use wsl paths in any configuration or script.
- never try to run commands wsl terminal that interact with codacy cli.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mzfshark) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-13 -->
