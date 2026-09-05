# LandingZone-IaC — Copilot / Agent Instructions

Authoritative context for AI agents and contributors. These rules derive from
`.specify/memory/constitution.md` (v1.1.0). The constitution wins on any conflict.

## What this repo is
An Azure Landing Zone delivered entirely as Infrastructure as Code, grown one
capability at a time, reproducibly deployable across multiple environments and
multiple Azure subscriptions with **zero manual drift**.

## Non-negotiable guardrails (from the constitution)
- **GitOps / source of truth**: Git is the single source of truth. All changes
  flow through reviewed pull requests. NEVER change live environments directly or
  via the portal (no click-ops) in managed scope.
- **Reproducibility & parity**: Identical code across all environments/
  subscriptions. Differences are expressed ONLY as declared configuration —
  never divergent code paths or branches per environment.
- **Idempotence**: Re-running a deploy against unchanged desired state MUST
  produce no changes.
- **Incremental & reversible**: Every change is small, independently deployable,
  and has a defined rollback/roll-forward strategy.
- **Security by default**: No secrets in code — ever. Least-privilege access by
  default. All resources auditable with diagnostic settings enabled.
- **Automation-first**: Everything is buildable by humans and agents
  interchangeably; operations are deterministic and machine-readable.
- **Documentation is part of "done"**: Update module docs, decision records
  (ADRs), and runbooks in the SAME pull request as the change.

## Baseline technology decisions
- **Dual IaC engines**: Support BOTH Bicep and Terraform. Each capability exposes
  an equivalent, standardized module interface (inputs/outputs/naming/tags) across
  both engines. Engine choice is declared configuration. Parity is a testable
  requirement — keep the two paths behaviorally identical.
- **Azure Verified Modules (AVM) first**: Compose from AVM modules wherever one
  exists, via specialized single-responsibility agents. Pin module versions.
  Custom modules only where no suitable AVM module exists, and only with written
  justification.
- **CI/CD & GitOps**: GitHub Actions/Workflows. PR runs plan/preview; promotion is
  lower-to-higher environment. Authenticate to Azure with **federated OIDC
  (workload identity)** — NO long-lived credentials in the repo.
- **Private by default**: Every resource is within-VNet — private endpoints,
  private DNS, no public exposure unless an explicit, reviewed, justified
  exception is declared in configuration.
- **Containers**: AKS OR Azure Container Apps (ACA), selectable by configuration,
  both private by default.
- **API governance**: Azure API Management (APIM) is the default policy enforcement
  point, but keep it behind a stable, documented gateway interface so an
  alternative (e.g., Kong/KGateway) can be substituted without changing dependents.
- **External access**: Azure Front Door OR Application Gateway (selectable), with
  WAF enabled by default.
- **Identity & secrets**: Managed identities preferred; Microsoft Entra RBAC with
  least privilege; all secrets/certs in Azure Key Vault (referenced, never inlined).
- **Observability**: Central Log Analytics + Azure Monitor + Application Insights;
  diagnostic settings on by default for every resource.
- **Python**: Used where needed for tooling/validation/glue. Provide Python
  Notebooks for onboarding, health checks, and post-deploy verification.

## Module contract (every capability)
Each module MUST declare and document:
- **Purpose**, **inputs**, **outputs**, **dependencies**, **assumptions**,
  **usage examples**.
- Explicit interface — no hidden cross-module dependencies, no hardcoded
  environment- or subscription-specific values (pass them as inputs).
- Independently versioned, independently testable, composable without modification.

## Naming & tagging
- Apply the single standardized naming convention uniformly across Bicep and
  Terraform.
- Apply the mandatory tag set (including cost-attribution tags) to every resource.

## Validation gates — what "green" means
A change is mergeable only when ALL pass together:
1. **Static validation** (format/lint/static analysis) — blocking.
2. **Policy/compliance checks** (e.g., Azure Policy) — blocking.
3. **Plan/preview review** — the plan shows ONLY intended changes, zero
   unexplained diffs (verify for both Bicep and Terraform paths where applicable).
4. **Post-deploy verification** — asserts observable outcomes (existence,
   configuration, health), validated in a lower environment before promotion.

## Do / Don't
- DO keep changes small, reviewable, and traceable to a spec/plan/task.
- DO record architecturally significant decisions as versioned ADRs.
- DO flag unknowns as `[NEEDS CLARIFICATION]` in specs.
- DON'T introduce dead or commented-out code.
- DON'T add public endpoints, secrets, or standing broad privileges by default.
- DON'T create per-environment code divergence — use configuration.
- DON'T skip the validation gate or promote on a failed/absent gate.

## Working with Spec Kit
- Constitution: `.specify/memory/constitution.md`.
- Flow per capability: `/speckit.specify` → `/speckit.clarify` → `/speckit.plan`
  → `/speckit.tasks` → `/speckit.implement`.
- Capability dependency order: Identity → Networking → Observability →
  Containers → Edge/API. Reference earlier capabilities' outputs; don't redefine.

## Active Technologies
- Bicep (latest CLI) and Terraform HCL (>= 1.9) + Azure Verified Modules (AVM) (Bicep Registry + Terraform AVM) (000-platform-baseline)
- Terraform remote state in Azure Storage with blob-lease locking (000-platform-baseline)
- Python 3.11+ for validation tooling, health-check notebooks, and post-deploy verification (001-identity-secrets-access)
- Azure CLI (`az`) and GitHub Actions `azure/login` with federated OIDC; Microsoft Entra ID for groups and identity federation (001-identity-secrets-access)
- Bicep (latest CLI) and Terraform HCL (>= 1.9); Python 3.11+ for validation, health-check notebooks, and post-deploy verification. + Azure Verified Modules (Bicep Registry AVM + Terraform AVM) for virtual network, subnets/NSGs, private DNS zones, private endpoints, Azure Firewall + firewall policy, and route tables; `azurerm` + `azapi` Terraform providers; Azure CLI (`az`); Microsoft Azure Monitor (central Log Analytics) for diagnostics and Traffic Analytics; Network Watcher for NSG flow logs. (002-core-networking)
- No application data. Terraform remote state in Azure Storage with blob-lease locking (inherited from Platform Baseline). No secrets managed by this capability. (002-core-networking)
- Bicep (latest CLI) and Terraform HCL (>= 1.9); Python 3.11+ for validation, KQL health-check/onboarding/post-deploy notebooks, and verification. + Azure Verified Modules (Bicep Registry AVM + Terraform AVM) for Log Analytics workspace, Application Insights (workspace-based component), diagnostic settings, action groups, metric/scheduled-query alert rules, and workbooks; `azurerm` + `azapi` Terraform providers; Azure CLI (`az`); Azure Monitor / Log Analytics query API (KQL) consumed by the notebooks via the Azure SDK for Python. (003-observability-monitoring)
- Log Analytics workspace (telemetry store, config-driven retention). Terraform remote state in Azure Storage with blob-lease locking (inherited from Platform Baseline). No secrets managed by this capability; Application Insights connection info is exposed as a Key Vault reference, never inlined. (003-observability-monitoring)
- Bicep (latest CLI) and Terraform HCL (>= 1.9); Python 3.11+ for validation, posture/parity gates, and post-deploy verification. + Azure Verified Modules (Bicep Registry AVM + Terraform AVM) for AKS managed cluster (`avm/res/container-service/managed-cluster`), ACA managed environment (`avm/res/app/managed-environment`), and user-assigned managed identity (`avm/res/managed-identity/user-assigned-identity`); role assignment for `AcrPull` on the consumed registry; `azurerm` (`~> 4.0`) + `azapi` Terraform providers; Azure CLI (`az`). Consumes — does not provision — the shared private container registry, Core Networking seams (subnets/private DNS), the identity model (001), and the observability backbone (003). (004-apps-container-platform)
- No application data. Terraform remote state in Azure Storage with blob-lease locking (inherited from Platform Baseline). No secrets managed by this capability; where a secret is unavoidable it is a Key Vault reference (capability 001), never inlined. (004-apps-container-platform)
- Bicep (latest CLI) and Terraform HCL (>= 1.9); Python 3.11+ for validation, posture/parity gates, notebooks, and post-deploy verification. + Azure Verified Modules (pinned) — Bicep `avm/res/cognitive-services/account` `0.15.1` (Foundry account = `AIServices` kind, projects as child resources, model `deployments`, private endpoints, diagnostics, managed identity), `avm/res/managed-identity/user-assigned-identity` `0.6.0` (+ `.../federated-identity-credential` `0.2.0`); Terraform `Azure/avm-res-cognitiveservices-account/azurerm` and `Azure/avm-res-managedidentity-userassignedidentity/azurerm` (pinned in `capability.yaml`/`versions.md`). Direct (non-AVM) only where no module wraps it: Entra **role assignments** (least-privilege RBAC for MI→Foundry/data), and the model-deployment/project child config. `avm/ptn/ai-ml/ai-foundry` `0.7.0` was evaluated and **rejected as the primary** (it provisions its own networking/storage rather than consuming the landing-zone seams — see research R1). The **gateway** (APIM AI Gateway, `avm/res/api-management/service`) and **grounding data stores** (`avm/res/search/search-service`, Storage) are **consumed, not provisioned** here. `azurerm` (`~> 4.0`) + `azapi` Terraform providers; Azure CLI (`az`); Azure AI/OpenAI SDK for Python (notebooks) invoking through the gateway seam. (005-ai-platform-foundry)
- No application data owned here. Grounding data sources are consumed as declared, private-endpoint-only dependencies (FR-009). Terraform remote state in Azure Storage with blob-lease locking (inherited from Platform Baseline). No secrets managed by this capability; any unavoidable secret/connection info is a Key Vault reference (capability 001), never inlined. (005-ai-platform-foundry)
- Bicep (latest CLI) and Terraform HCL (>= 1.9); Python 3.11+ for validation, posture/parity gates, notebooks, and post-deploy verification. + Azure Verified Modules (pinned). Bicep — `avm/res/network/application-gateway` `0.10.0` (default edge; WAF_v2, TLS listeners, backend pools), `avm/res/network/application-gateway-web-application-firewall-policy` `0.3.0` (App Gateway WAF policy), `avm/res/cdn/profile` `0.20.0` (Front Door Standard/Premium alternative edge; AFD endpoint/origin-group/route/custom-domain/security-policy children), `avm/res/network/front-door-web-application-firewall-policy` `0.3.3` (Front Door WAF policy), `avm/res/api-management/service` `0.14.4` (gateway; internal VNet mode by default), `avm/res/managed-identity/user-assigned-identity` `0.6.0`. Terraform — `Azure/avm-res-network-applicationgateway/azurerm`, `Azure/avm-res-network-applicationgatewaywebapplicationfirewallpolicy/azurerm`, `Azure/avm-res-cdn-profile/azurerm`, `Azure/avm-res-network-frontdoorwebapplicationfirewallpolicy/azurerm`, `Azure/avm-res-apimanagement-service/azurerm`, `Azure/avm-res-managedidentity-userassignedidentity/azurerm` (versions pinned in `capability.yaml`/`versions.md`). Direct (non-AVM) only where no module wraps it: Entra **role assignments** (least-privilege RBAC for MI→backends/Key Vault) and the gateway↔backend routing wiring. The **private endpoints / private DNS / subnets** (002), the **AKS/ACA ingress seam** (004), and any **AI model/agent backend seam** (005) are **consumed, not provisioned** here. `azurerm` (`~> 4.0`) + `azapi` Terraform providers; Azure CLI (`az`); Python `requests`/Azure SDK for notebook-based north-south verification through the edge. (006-edge-api-gateway)
- No application data owned here. No API definitions/products owned here (Non-Goal). Terraform remote state in Azure Storage with blob-lease locking (inherited from Platform Baseline). No secrets managed by this capability; TLS certificates and any unavoidable connection info are Key Vault references (capability 001), never inlined. (006-edge-api-gateway)

## Recent Changes
- 000-platform-baseline: Added Bicep (latest CLI), Terraform HCL (>= 1.9), Python 3.11+ + Azure Verified Modules (Bicep Registry + Terraform

---
> Source: [pkumar26/LandingZone-IaC](https://github.com/pkumar26/LandingZone-IaC) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-04 -->
