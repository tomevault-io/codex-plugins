# Copilot Instructions for vulnspringbootapp

## Project Purpose
This is an intentionally vulnerable Spring Boot application (baseline Spring Boot v3.1.5) designed for testing security tools like Dependabot, Renovate, CodeQL, and dependency review workflows. The codebase deliberately contains known vulnerabilities. Spring Boot 3.1.5 is chosen as the earliest release with official Java 21 support while still carrying known transitive CVEs.

## Architecture Overview
- **Minimal Spring Boot app**: Single REST controller with one endpoint
- **Package structure**: `com.example.vulnspringbootapp` (flat, no layers)
- **No persistence**: In-memory only, no database
- **No tests**: Testing infrastructure intentionally omitted

## Build System: Gradle with Kotlin DSL

### Version Management
All versions are in `gradle.properties`:
- `springBootVersion=3.1.5` (**DO NOT UPDATE FURTHER** - intentionally vulnerable baseline)
- `springDependencyManagementVersion=1.1.6`
- `javaVersion=21`

Build system uses:
- Gradle 8.5 (supports JDK 17-21)
- JDK 21 for CI/CD workflows

### Build Commands
```bash
./gradlew clean build    # Build the application
./gradlew bootRun        # Run locally (http://localhost:8080)
```

## Critical Rules

### Security Vulnerabilities Are Intentional
- **NEVER** update Spring Boot version beyond 3.1.5 or fix security vulnerabilities
- **NEVER** add security hardening (authentication, input validation, etc.)
- This app exists to trigger Dependabot alerts and test dependency review workflows
- Expect security scanners to report issues—this is the desired state

### Keep It Simple
- No service layer, repositories, or DTOs
- No test coverage
- Only direct dependencies: `spring-boot-starter-web` and one intentionally vulnerable library (e.g. `commons-text:1.9`)
- Single controller with minimal logic

## Dependency Management & CI/CD

### Dependabot Configuration
- Updates only **direct** dependencies (not transitive)
- Runs weekly, max 10 open PRs
- Will create PRs to fix vulnerable dependencies (intentionally ignored/closed)

### Dependency Review Workflow
**Purpose**: Block PRs with high/critical vulnerabilities and add explanatory comments

Configuration in `.github/workflows/dependency-review.yml`:
- Fails on `high` or `critical` severity vulnerabilities
- Always adds PR comment with vulnerability details
- Requires dependency graph enabled (and GHAS for private repos)

### CI Workflow
- Runs on push/PR to main/master/develop
- Uses JDK 21 with Amazon Corretto
- Command: `./gradlew clean build --no-daemon`

### Dependency Submission Workflow
- Uses Gradle 8.5 with JDK 21
- Submits dependency graph via `gradle/actions/setup-gradle@v4`

## Code Structure

### Application Entry Point
```java
@SpringBootApplication
public class VulnSpringBootAppApplication {
    public static void main(String[] args) {
        SpringApplication.run(VulnSpringBootAppApplication.class, args);
    }
}
```

### Controller Pattern
```java
@RestController
public class HelloController {
    @GetMapping("/")
    public String hello() {
        return "Hello, World!";
    }
}
```

## When Adding Features
- Add controllers directly in root package
- Keep endpoints simple (no business logic)
- Do not add security measures
- Avoid adding dependencies unless absolutely necessary
- Maintain the vulnerable state (intentionally outdated/vulnerable versions)
    - If a dependency is upgraded by automated tooling and reduces vulnerability coverage, prefer reverting unless demonstrating an upgrade scenario.

## Code Simplicity Rules
- Prefer the smallest change that achieves the goal.
- Avoid abstractions, layers, or patterns unless they reduce code size and clarify behavior.
- **Always** keep functions short and explicit; return early on non-happy paths.
- Favor standard library over third-party where practical.
- Comments: Only add in-code comments when behavior is non-obvious or unexpected. Code should be simple enough to make the "what" clear; comments should explain "why" we are doing it this way, not restate "how".

## Python Guidelines (scripts/auto_merge.py)
- Dependencies: use `PyGithub` only; install via `pip install PyGithub` in workflows.
- Logging: use the `logging` stdlib with `LOG_LEVEL` env to control verbosity (default `INFO`; allow `DEBUG`).
- Inputs: read Actions event payload from `GITHUB_EVENT_PATH`; prefer PR author and head ref over `github.actor`.
- Configuration precedence: `workflow_dispatch.inputs.compat_threshold` → `DEPENDABOT_COMPAT_THRESHOLD` env → `DEFAULT_COMPAT_THRESHOLD` env → fallback `80`.
- Outputs: write plain `key=value` to `GITHUB_OUTPUT`; append brief lines to `GITHUB_STEP_SUMMARY`.
- Robust parsing: support Dependabot title formats (optional backticks, 2- or 3-part versions, optional leading `v`).
- Decisions: allow auto-merge for patch; for minor only if compatibility score ≥ threshold; otherwise request manual review.
- Idempotence: use comment markers to avoid duplicate postings; never fail the workflow on comment errors.
- Environment: require `GITHUB_TOKEN`; do not read secrets beyond what the workflow passes.
- Keep functions single-responsibility (parse, compute, comment) and avoid classes.
- Functions shouldn't exceed ~30 lines; break down complex logic.

## GitHub Actions Guidelines (Workflows)
- Triggers: use `pull_request_target` for PRs (Dependabot requires this) and `workflow_dispatch` for manual runs.
- Permissions: set least privilege (`contents: write`, `pull-requests: write`); avoid broad default permissions.
- Checkout: always include `actions/checkout@v4` before running repo scripts.
- Python: use `actions/setup-python@v5`; pin to `'3.x'` and `pip install PyGithub`.
- Conditions: gate jobs on PR author/login and branch prefix (`dependabot/`), not `github.actor`.
- Dispatch inputs: pass `compat_threshold` via step `env`; let the script handle defaults (avoid `||` expressions in YAML).
- Step ordering: disable auto-merge first (clean slate) then decide and optionally enable native auto-merge.
- Auto-merge: prefer native enabling via `peter-evans/enable-pull-request-automerge@v3`; guard with `MERGE_ALLOWED` and ensure a valid PR number. Use `disable-pull-request-automerge@v3` at job start.
- Runner efficiency: do not hold runners waiting for checks; enable native auto-merge and exit.
- Summary: write concise decision summaries; avoid verbose logs in the summary.

## Simplicity for Expressions
- Avoid complex GitHub expressions in `env:` (e.g., `||`). Instead, pass raw inputs and handle fallback in Python.
- Use `startsWith(...)` and direct event fields (`github.event.pull_request.user.login`, `github.event.pull_request.head.ref`).
- Prefer explicit `pull-request-number` when using PR actions to prevent context ambiguity.

## Non-Goals
- Do not add tests or CI stages beyond what’s necessary for dependency review workflows.
- Do not add security hardening; vulnerabilities are intentional for tooling validation.

---
> Source: [faccenda-org/vulnspringbootapp](https://github.com/faccenda-org/vulnspringbootapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-22 -->
