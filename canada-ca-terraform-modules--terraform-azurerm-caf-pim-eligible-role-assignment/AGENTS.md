<!-- spectre: copilot-only -->

# Copilot Instructions for terraform-azurerm-caf-pim_eligible_role_assignment

## Module behavior to preserve
- `scope` is a `list(string)` and should contain full Azure resource IDs.
- `principal_id` is consumed as a list in locals and assignment expansion logic.
- `basename(scope)` is used for Terraform resource key naming only, not as Azure scope input.

## Validation and lint workflow
- For this module repo, run validation with backend disabled:
  - `terraform init -backend=false`
  - `terraform validate`
- Validate both directories when making changes:
  - repo root
  - `test/`
- Run lint recursively from root:
  - `tflint --recursive`
- Skill for this workflow:
  - **terraform-validation-workflow** (local: `.github/skills/terraform-validation-workflow/SKILL.md`; fallback: `~/.copilot/skills/terraform-validation-workflow/SKILL.md`)

## Version constraints
- Keep Terraform and provider constraints in `versions.tf` files:
  - root `versions.tf` should define `required_version` and `required_providers.azurerm`
  - `test/versions.tf` should define `required_version`
- Exception: keep ESLZ example `required_version` in `ESLZ/PimEligibleRoleAssignment.tf` (do not reintroduce `ESLZ/versions.tf`).
- Skill for versioning guidance:
  - **terraform-versioning-advisor** (local: `.github/skills/terraform-versioning-advisor/SKILL.md`; fallback: `~/.copilot/skills/terraform-versioning-advisor/SKILL.md`)

## Repository hygiene
- Do not commit generated Terraform working directories:
  - `.terraform/` (any level)
- Keep `.gitignore` rules aligned with module-repo expectations.
- Keep `ESLZ/PimEligibleRoleAssignment.tfvars` tracked as an example file.
- Keep `ESLZ/` as a user-facing implementation example; use `test/` for local module validation.

## Documentation expectations
- README examples must match actual variable types (`scope` list, `principal_id` list).
- Prefer minimal, targeted changes and keep examples consistent with module behavior.
- Skill for docs workflow:
  - **terraform-docs-workflow** (local: `.github/skills/terraform-docs-workflow/SKILL.md`; fallback: `~/.copilot/skills/terraform-docs-workflow/SKILL.md`)

## Resource design and naming
- Use simple, stable names for Terraform block identifiers (`this`, `main`, `default`)
- Compose actual resource names from variables using `locals` to ensure consistency and enforce provider constraints
- Never hardcode environment or region in module code; these must be inputs
- Skills for naming conventions:
  - **terraform-naming-conventions** (local: `.github/skills/terraform-naming-conventions/SKILL.md`; fallback: `~/.copilot/skills/terraform-naming-conventions/SKILL.md`) — provider-agnostic
  - **ssc-azure-naming** — `~/.copilot/skills/ssc-azure-naming/SKILL.md` (global only; if using Government of Canada / SSC constraints)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/canada-ca-terraform-modules)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/canada-ca-terraform-modules)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
