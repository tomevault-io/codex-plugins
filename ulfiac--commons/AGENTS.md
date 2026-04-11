# Copilot Instructions for commons

## Repository Overview
This repository, `commons`, serves as a centralized hub for reusable GitHub Actions workflows. It provides standardized, maintainable CI/CD components that can be shared across multiple repositories within the ulfiac organization.

## Key Components
- **Reusable Workflows**: Located in `.github/workflows/`, these are designed to be called from other repositories using `workflow_call`.
- **Primary Workflows**:
  - `reusable_linter.yaml` - A comprehensive linting workflow that performs multiple checks:
    - GitHub Actions workflow linting (actionlint)
    - Shell script linting (shellcheck)
    - Terraform formatting validation (terraform fmt)
    - Terraform static analysis (tflint)
    - HCL formatting validation (terragrunt hcl fmt)
    - Security scanning of configuration files (trivy config)
  - `reusable_purge_workflow_logs.yaml` - A workflow log cleanup utility that:
    - Deletes workflow runs older than a specified retention period (default: 7 days)
    - Uses the GitHub CLI (`gh`) for log management
    - Requires `actions: write` permission

## Interrelations with Other Repositories
This repository is used by several other repositories in the workspace as a shared dependency:

- **aws-bootstrap**: Uses the reusable linter workflow to validate Terraform configurations and scripts during CI/CD.
- **aws-infrastructure**: Leverages the linting workflow for its Terraform code and shell scripts.
- **github-repos**: Utilizes the workflow for maintaining code standards in its Terraform configurations.
- **vscode-workspaces**: While primarily a workspace configuration, it may benefit from the shared workflows if expanded.

## Usage Pattern
Other repositories reference these workflows using:
```yaml
# Linting workflow
uses: ulfiac/commons/.github/workflows/reusable_linter.yaml@main

# Log purge workflow
uses: ulfiac/commons/.github/workflows/reusable_purge_workflow_logs.yaml@main
with:
  retain-days: 7
```

## Development Guidelines
- Keep workflows generic and configurable through inputs.
- Test changes locally and in dependent repositories before merging.
- Update this instruction file when adding new reusable workflows or significant changes.
- Update the repository's README with new workflow documentation

## Security Considerations
- Workflows run with minimal permissions by default (contents: read).
- The purge workflow requires `actions: write` permission to delete workflow runs.
- Use pinned action versions for reproducibility and security.
- Regularly update dependencies and review security advisories.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ulfiac)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/ulfiac)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
