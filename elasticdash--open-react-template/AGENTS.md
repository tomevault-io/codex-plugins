# Copilot Execution Contract (Anti-Stall)

## CORE RULE (NON-NEGOTIABLE)

You **cannot work in the background**.

If a message does **not** contain concrete, user-visible output
(code, schema, file content, or an explicit failure reason),
**NO WORK is considered done.**

Any claim of progress **without output** is a violation.

---

## MANDATORY WORKFLOW

### 1. PLAN (REQUIRED)

* Create `.temp/plan.md` if missing
* Status must be `PENDING_APPROVAL`
* Plan MUST include:

  * Summary
  * Checklist (atomic, verifiable steps)
  * Files affected
  * Risks
  * Complexity

### 2. APPROVAL

* Wait for **explicit approval**
  (`yes | approve | go ahead | proceed | ok | confirm`)
* No execution before approval

### 3. EXECUTION

* Execute **EXACTLY ONE checklist step per message**
* That message MUST contain the **actual output** of the step
* No output = step **not started**

After producing output:

* Mark step `[x]` in `plan.md`
* Update `Last Updated`
* Explicitly state: `Step N completed`

---

## CHECKLIST RULES (STRICT)

Each checklist item MUST:

* Represent one atomic action
* Define expected concrete output
* Be verifiable by inspecting the output

❌ Invalid: "Improve API schema"
✅ Valid: "Output full response schema JSON for POST /pokemon/search with field-level descriptions"

---

## FAILURE / BLOCKED RULE

If a step cannot be completed:

* Explain **exactly** what is missing
* Stop execution immediately
* Do NOT mark the step complete
* Do NOT defer to a future message

---

## FORBIDDEN BEHAVIOR (ANTI-STALL)

You MUST NOT:

* Claim progress without output
* Say "working on it", "in progress", "please wait"
* Defer execution to a later message
* Mark a step complete without visible results

---

## RESUME RULE

On "resume" / "continue":

* Read `.temp/plan.md`
* Report:

  * Original task
  * Completed steps `[x]`
  * Remaining steps `[ ]`
* Execute the **next unfinished step immediately**
  OR fail fast with a blocking reason

---

## TECH RULES

* Next.js 13+: use `next/navigation`
* All API calls under `./services/`
* TypeScript only, fully type-safe
* Import shared functions as:
  `import { fn } from "@/services/file"`

---

## CODING STANDARDS (ENFORCED)

### Language & Type Safety

* **Language:** TypeScript only
* Use strict type safety and explicit types
* Use `unknown` instead of `any` where input is not validated
* Assert types explicitly with guards
* Avoid untyped `any` usage

### Naming Conventions

* Types and interfaces: **PascalCase**
* Functions and variables: **camelCase**
* Constants: **UPPER_CASE_WITH_UNDERSCORES**
* Tests: describe structure matching file under test

### Code Style

* Prefer **functional, declarative coding** patterns where applicable
* Favor composable utilities over large monolithic functions

### Architecture (MANDATORY)

* Enforce separation of concerns:
  * API layer
  * Business logic / domain
  * Infrastructure / service integration

### Documentation (REQUIRED)

* All functions MUST have JSDoc comments describing intent and side effects
* Generated code must have clear comments explaining logic
* All generated files MUST include header comments describing generation source and purpose
* New modules require an entry in project documentation (`README.md` or docs folder)

### Error Handling (STRICT)

* All API functions MUST return structured error responses
* No untyped error objects
* Produce explicit failure reason when blocked

### File Organization

* All new API routes go under `./services/`
* Use `next/navigation` utilities in Next.js components
* Shared utilities must be imported as:
  ```
  import { fn } from "@/services/file"
  ```

### Testing (MANDATORY)

* Unit tests are REQUIRED for all new functions
* Use the project's testing framework (e.g., Vitest, Jest)
* Tests MUST assert:
  * Happy path
  * Error paths
  * Edge cases
* Include test output or errors in your response when relevant

---

## OUTPUT REQUIREMENTS

### For Every Task Step:

* Provide full file content when creating or updating a file
* Include code blocks with language annotation (e.g., ```ts)
* Include test output or errors when tests are relevant

### For Failure States:

* Produce explicit failure reason
* State exactly what input is missing
* Do NOT defer or claim partial progress

---

## FINAL GUARANTEE

Execution **only exists when output exists**.
If nothing is produced, nothing happened.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ElasticDash) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
