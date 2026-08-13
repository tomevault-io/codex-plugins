# MMO FES Function App - AI Coding Agent Instructions

## Project Overview
Azure Functions v2 timer-triggered app for DEFRA MMO (Marine Management Organisation). Replaces CRON jobs to orchestrate scheduled tasks for the `mmo-fes-reference-data-reader` service and reconcile certificate data with a business continuity system.

**Two Azure Functions:**
- `mmo-fes-functionapp`: Timer-triggered HTTP orchestrator with retry logic (calls data reader every X hours)
- `mmo-fes-reconciliationapp`: HTTP-triggered MongoDB → Business Continuity API reconciliation

## Architecture & Patterns

### Retry Pattern with Exponential Backoff
Both functions implement custom retry logic with increasing delays:
```javascript
// Pattern: retry(fn, retries, delayFn) where delay = (totalRetries - retriesRemaining) * baseDelay
// Example: 5-min base delay → attempts at 0min, 0min, 5min, 10min, 15min
const calcDelay = (delay, totalRetries) => (retriesRemaining) =>
  (totalRetries - retriesRemaining) * delay;
```

### Config via Environment Variables
All functions load config from env vars with fallback defaults in the module-level `config` object. Use `overrideConfig` parameter in tests:
```javascript
await func(context, myTimer, { retries: 2, retryDelay: 1000 });
```

### Application Insights Integration
Custom correlation with `ai.operation.id` from Azure context. Always initialize AppInsights first, then axios interceptors:
```javascript
appInsights.init(config.instrumentationKey, context);
axiosInterceptors.init(axios);  // Adds duration tracking to responses
```

### HTTPS Agent with Custom CA Bundle
Loads `cabundle.pem` from function directory for custom certificate chains. Located at `${context.executionContext.functionDirectory}/../cabundle.pem`.

## Development Workflow

### Local Development
```bash
# Prerequisites: Azure Functions Core Tools v3+, Azurite storage emulator
npm install
azurite  # Run in separate terminal
func start  # Starts both functions
```

### Testing
```bash
npm test              # Run tests with coverage
npm run test:ci       # CI mode with jest-junit reporter
```

**Test Conventions:**
- Use `jest.spyOn()` for mocking, never reassign `require()`
- Mock MongoDB via `__mocks__/mongodb.js` (configured in `package.json` moduleNameMapper)
- Mock `setTimeout` to execute callbacks immediately: `jest.spyOn(global, 'setTimeout').mockImplementation((callback) => callback())`
- Context structure: `{ log: jest.fn(), executionContext: { functionDirectory: __dirname }, traceContext: {}, operationId: '' }`

**Coverage Requirements (enforced):**
- Branches: 90%, Functions: 90%, Lines: 90%, Statements: 90%

## Project-Specific Conventions

### Logging Format
Structured logs with bracketed tags:
```javascript
context.log('[SCHEDULED-JOBS][LANDING-AND-REPORTING][STARTED]', timeNow());
context.log(`[SCHEDULED-JOBS][BC-RECONCILIATION][CONFIG][url: ${url}]`, timeNow());
```

### Function Structure
Every function exports a single async function with signature:
```javascript
const func = async (context, myTimer, overrideConfig) => {
  // 1. Log start, merge overrideConfig
  // 2. Check myTimer.IsPastDue
  // 3. Initialize AppInsights + interceptors
  // 4. Execute main logic with retry pattern
  // 5. Track events and log completion
};
module.exports = func;
```

### Timer vs HTTP Triggers
- `mmo-fes-functionapp`: Timer trigger (`timerTrigger` binding) with `CRONTIME` env var
- `mmo-fes-reconciliationapp`: HTTP trigger (`httpTrigger` + `http` output binding)

## Key Files & Directories

- `mmo-fes-functionapp/index.js`: Main timer function with HTTP retry orchestration
- `mmo-fes-reconciliationapp/index.js`: MongoDB → BC API reconciliation with batch processing
- `src/appInsights.js`: Shared AppInsights wrapper with operation correlation
- `src/axiosInterceptors.js`: Request/response duration tracking using `perf_hooks`
- `__mocks__/mongodb.js`: Jest mock for MongoDB (returns COMPLETE/VOID certificate fixtures)
- `function.json`: Azure Functions binding definitions (timer schedule, HTTP triggers)
- `host.json`: Defines both functions in `functions` array, enables AppInsights live metrics

## CI/CD & Deployment

### GitFlow Branching
**Strictly enforced** - pipelines fail on non-standard branch names:
- `main`: Production releases
- `develop`: Integration branch
- `feature/*`, `epic/*`: Feature development
- `hotfix/*`: Production fixes

### Azure Pipelines
Uses shared template from `mmo-fes-pipeline-common` repo:
```yaml
# azure-pipelines.yml extends shared template
parameters:
  - deployFromFeature: false  # Override to deploy feature branches
  - skipPRE1: false            # Skip PRE1 environment
```

### Docker Build
Node 22 runtime on Azure Functions v4:
```dockerfile
FROM mcr.microsoft.com/azure-functions/node:4-node24
# Includes githash tracking via ARG GIT_HASH
```

## Common Pitfalls

1. **MongoDB Mock Not Applied**: Ensure `moduleNameMapper` in `package.json` points to `__mocks__/mongodb.js`
2. **Axios Import Path**: Use `axios/dist/node/axios.cjs` in Jest moduleNameMapper to avoid ESM issues
3. **Timer Past Due**: Always check `myTimer.IsPastDue` and log when function execution is delayed
4. **CA Bundle Missing**: Functions fail silently if `cabundle.pem` not found; wrap in try-catch
5. **Retry Delay Calculation**: First retry is immediate (delay = 0), subsequent delays increment linearly

## Environment Variables (Per-Function Config)

### mmo-fes-functionapp
- `CRONTIME`: Cron schedule (e.g., `"15 1,12,13 * * *"` = 1am, 12pm, 1pm)
- `DATA_READER_URL`: Target endpoint (default: `http://localhost:9000/v1/jobs/landings`)
- `TIMEOUT_IN_MS`: HTTP timeout (default: 600000 = 10min)
- `NUMBER_OF_RETRIES`: Retry attempts (default: 4)
- `RETRY_DELAY_IN_MS`: Delay increment per retry (default: 300000 = 5min)

### mmo-fes-reconciliationapp
- `DB_NAME`, `DB_CONNECTION_URI`: MongoDB connection
- `BATCH_CERTIFICATES_NUMBER`: Batch size for BC API calls (default: 1000)
- `QUERY_START_DATE`, `QUERY_END_DATE`: Date range for certificate query
- `BUSINESS_CONTINUITY_URL`, `BUSINESS_CONTINUITY_KEY`: BC API credentials

## Node Version Requirements
- **Engine**: Node >=24.0.0 <25.0.0, npm ~10.9.2
- Lock to Node 22 runtime in Dockerfile and local development

## Standards precedence (highest wins)

When guidance conflicts, follow this order:

1. **DEFRA Software Development Standards** (mandatory) — https://defra.github.io/software-development-standards/
2. **DEFRA Digital Service Manual** — https://digital.defra.gov.uk/service-manual
3. **GOV.UK Service Standard & Service Manual (GDS)** — https://www.gov.uk/service-manual
4. **Community best practice** — [OWASP Secure Coding Practices](https://owasp.org/www-project-secure-coding-practices-quick-reference-guide/), [12-factor](https://12factor.net/), widely-adopted Node.js/Azure Functions patterns

> **DEFRA takes precedence over GDS. GDS takes precedence over community guidance.** Any deviation from a DEFRA standard MUST be raised as a formal exception through DEFRA's architectural governance (Delivery Architecture team: `delivery.architecture@defra.gov.uk`).

## The working framework (Triage → Read → Research → Plan Handoff → Plan Validation Research → Approval → Implement → Test → Iterate → Summarise)

This section is the **single source of truth** for the working loop. The custom agents ([Orchestrator](.github/agents/function-app-orchestrator.agent.md), [Planner](.github/agents/function-app-planner.agent.md), [Developer](.github/agents/function-app-developer.agent.md) and [Reviewer](.github/agents/function-app-reviewer.agent.md)) reference it and **must not restate or fork it**.

**Triage first — pick the right path by size and risk:**

- **Trivial / low-risk** (typo, comment/doc tweak, a small localised change with no impact on trigger bindings, retry logic, App Insights correlation, MongoDB operations, security or data correctness): skip the planner and heavy research. Do a light **Read → Implement → Test → Summarise**, and research only the specific point that is genuinely uncertain.
- **Non-trivial** (new trigger, retry/backoff change, App Insights instrumentation, MongoDB batch operations, security, or anything affecting data correctness or risky): run the full loop below.

Non-trivial loop:

1. **Read** — Read the relevant files/config in the repo for context before acting. Never assume; verify.
2. **Research** — Do thorough, risk-scoped research in the open and validate findings against DEFRA/GDS and Azure/framework guidance so advice reflects current APIs and policy. Cite sources.
3. **Clarify** — Ask the user targeted questions whenever requirements are ambiguous or missing. Surface requirement gaps explicitly with suggested fixes. Do not guess at intent.
4. **Plan handoff** — Delegate planning to the [Planner - Function App](.github/agents/function-app-planner.agent.md) agent. The planning agent returns the complete implementation plan.
5. **Plan validation research** — Perform thorough research in the open to validate the plan against DEFRA/GDS and Azure Functions guidance, **focusing on the steps the planner flagged as risky or version-sensitive** (unfamiliar APIs, retry patterns, security, policy). Send targeted revisions back to the planner.
6. **Approval** — Present the complete validated plan to the user and obtain explicit approval before implementation. If changes are requested, update the plan, re-validate, and re-approve. **Cap the plan → validate → approve → implement replanning cycle at 3 iterations**; if it is still unresolved, stop and surface the blocker to the user.
7. **Implement** — Deliver one task at a time (or parallel independent tasks) from the approved plan. Stay focused on the requested outcome; do not scope-creep or refactor unrelated code. When a change introduces or alters architecture, capture the decision as an ADR and update the relevant docs **where the repo already keeps them** (e.g. `docs/`).
8. **Test / Validate** — Run the test suite (`npm test`, or `npm run test:ci` in CI), check errors, and confirm each task works before moving on.
9. **Iterate** — Refine until the user is satisfied with each task.
10. **Summarise** — End with a detailed **executive summary** of what changed, why, how it was validated, and any follow-ups or risks.

## Workflow agents

Non-trivial work is coordinated through four custom agents that all run the framework above:

| Agent | Role |
|-------|------|
| [Orchestrator - Function App](.github/agents/function-app-orchestrator.agent.md) | Plans, delegates, verifies and reports; owns the Yes/No user-approval gate. Does **not** implement. |
| [Planner - Function App](.github/agents/function-app-planner.agent.md) | Internal planning subagent; produces the approval-ready plan and the research behind it. |
| [Developer - Function App](.github/agents/function-app-developer.agent.md) | Implements an already-approved plan end-to-end with tests. |
| [Reviewer - Function App](.github/agents/function-app-reviewer.agent.md) | Read-only review against DEFRA standards; reports findings by severity. |

Research (§4.2) and plan-validation research (§4.5) use the [deep-research-defra-alignment](.github/skills/deep-research-defra-alignment/SKILL.md) skill. The [Speckit](.github/agents) agents (`speckit.*`) are a separate spec-driven toolset and are **not** part of this workflow.

## Skills

Use `/develop` for implementation, coding, and research tasks. Use `/unit-tests` for writing tests, coverage, and SonarQube issues.

## Defra standards and governance

This service must comply with [Defra software development standards](https://github.com/DEFRA/software-development-standards) — the single source of truth. The rules below encode those standards; they do not replace them. When a standard changes, update this file.

### Quality gates

All code must pass these checks before merging:

- All tests pass (`npm test`, or `npm run test:ci` in CI)
- Coverage ≥90% (Branches/Functions/Lines/Statements) — no decrease from the SonarCloud baseline
- SonarQube/SonarCloud quality gate passes; security hotspots reviewed and resolved
- At least one approving review from another developer
- No unresolved security vulnerabilities in dependencies

### Security and PII

- Follow [OWASP Secure Coding Practices](https://owasp.org/www-project-secure-coding-practices-quick-reference-guide/)
- Never commit secrets — load all configuration and credentials from environment/App Settings (or Key Vault), never a populated `local.settings.json` in source
- **Never log PII**: names, addresses, emails, phone numbers, NI numbers, bank details, usernames, passwords, API keys, tokens — including in Application Insights telemetry and custom events
- Validate and sanitise all external input, especially on HTTP-triggered functions; use parameterised queries for database access
- Avoid `eval`, dynamic `Function()`, or executing user-supplied data; validate and normalise file paths

### Logging

- Structured logging with bracketed context tags and correlation propagated through Application Insights (operation/invocation ID)
- Levels: `error` (failures), `warn` (handled but unexpected), `info` (business events), `debug` (development only)

### Dependencies

- New dependencies must be widely used, actively maintained, and compatible with the current Node.js LTS
- Do not introduce a second HTTP client, database driver, or date library without an approved exception

### How Copilot should respond

- Follow conventions already in the codebase — check existing patterns first
- Prefer modifying existing files over creating new ones when the change fits naturally
- Provide minimal diffs touching only the necessary files; do not refactor unrelated code
- Always include or update tests for changed behaviour, mocking MongoDB, HTTP/Axios, and timers
- If a request conflicts with these instructions — a discouraged library, a skipped test, a hard-coded secret, or a broken quality gate — flag it explicitly and do not proceed silently

### Licence

All code is published under the [Open Government Licence v3.0](https://www.nationalarchives.gov.uk/doc/open-government-licence/version/3/) unless an approved exception exists.

<!-- STANDARDS NOTE: These instructions reflect Defra software development standards (https://github.com/DEFRA/software-development-standards). Review this file periodically or after any Defra standards update. -->

---
> Source: [DEFRA/eutd-mmo-fes-function-app](https://github.com/DEFRA/eutd-mmo-fes-function-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-13 -->
