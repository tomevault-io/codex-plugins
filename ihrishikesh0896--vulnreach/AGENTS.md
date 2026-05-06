Excellent.
That’s exactly how you build something real.

We’ll define:

> **VulnReach v2 – Phase 1**
> Tools: **Trivy** (SCA) + Internal **Tainter (static reachability engine)**
> Everything else deferred.

No ZAP.
No Syft wrapper complexity (we can let Trivy handle SBOM internally for now).
No Ollama initially (optional later in phase 1.5).
No dynamic runtime yet.

We build the deterministic correlation core properly first.

---

# 🛡 VulnReach v2 – Phase 1 TODO (Comprehensive)

This is structured in execution order.

---

# 🧱 0️⃣ Architecture Setup (Foundation)

## ✅ 0.1 Project Restructure

```
/api
/core
/agents
/storage
/correlation
/config
/tests
```

Define:

* Clear module boundaries
* No cross-import spaghetti
* Agent interface contract

---

## ✅ 0.2 Define Agent Interface

Create base class:

```python
class BaseAgent:
    async def run(self, context: ScanContext) -> AgentResult:
        ...
```

Define:

```python
class AgentResult:
    tool_name: str
    findings: list
    metadata: dict
```

Lock this contract early.

---

# 🌐 1️⃣ API Layer (FastAPI)

Using:
FastAPI

## ✅ 1.1 Create Scan Endpoint

```
POST /scan
```

Input:

```json
{
  "repo_path": "/path/to/project",
  "config_path": "/configs/scan.yaml"
}
```

Output:

```json
{
  "scan_id": "uuid",
  "status": "started"
}
```

---

## ✅ 1.2 Add Scan Status Endpoint

```
GET /scan/{scan_id}
```

Returns:

* current state
* completed agents
* risk summary (if done)

---

# 🗂 2️⃣ Config Engine

## ✅ 2.1 Define YAML Schema

Example:

```yaml
scan:
  static_reachability: true

risk:
  exposure: public
  data_sensitivity: high

policy:
  block_if:
    - severity: CRITICAL
      verdict: CONFIRMED
```

---

## ✅ 2.2 Validate YAML via Pydantic

Create strict config models.
Fail fast on invalid config.

---

# 🧠 3️⃣ Evidence Storage Layer

Using:
PostgreSQL

## ✅ 3.1 Define Core Tables

### scans

* id
* status
* created_at

### vulnerabilities

* scan_id
* package
* cve_id
* severity
* version
* fix_version

### reachability_evidence

* scan_id
* cve_id
* file
* function
* sink
* confidence

### correlation_results

* scan_id
* cve_id
* verdict
* risk_score
* priority

---

## ✅ 3.2 Store Raw Tool Output

Store:

* Full Trivy JSON
* Full Tainter JSON

For auditability.

---

# 🔎 4️⃣ Trivy Agent (SCA Agent)

Using:
Trivy

## ✅ 4.1 Wrap CLI Execution

* Run Trivy via subprocess
* Enforce timeout
* Capture stdout JSON

---

## ✅ 4.2 Normalize Output

Extract:

* package
* version
* CVE
* severity
* fixed version

Convert into internal model.

---

## ✅ 4.3 Edge Case Handling

* No vulnerabilities
* CLI failure
* Partial scan
* Unsupported ecosystem

---

# 🔬 5️⃣ Tainter Agent (Static Reachability)

This is your differentiator.

## ✅ 5.1 Detect Vulnerable Package Usage

For each Trivy CVE:

* Check if package is imported
* Check if vulnerable symbol used
* Identify call chain (if possible)

---

## ✅ 5.2 Static Sink Mapping

Map:

CVE → known vulnerable function
Then check if function reachable via taint flow.

---

## ✅ 5.3 Output Structured Evidence

```json
{
  "cve": "CVE-XXXX",
  "import_detected": true,
  "call_chain_exists": true,
  "sink_reachable": false,
  "files": [...]
}
```

---

# 🧮 6️⃣ Deterministic Correlation Engine

This is the core.

## ✅ 6.1 Define Reachability Logic

Initial logic:

```
IF import_detected AND call_chain_exists:
    verdict = LIKELY

IF import_detected AND sink_reachable:
    verdict = CONFIRMED

ELSE:
    verdict = NOT_OBSERVED
```

Keep simple for Phase 1.

---

## ✅ 6.2 Add Risk Score Formula (Minimal v1)

```
Base Severity Score:
CRITICAL = 4
HIGH = 3
MEDIUM = 2
LOW = 1

Reachability Multiplier:
CONFIRMED = 1.5
LIKELY = 1.2
POSSIBLE = 1.0
NOT_OBSERVED = 0.5

Risk = Severity × Multiplier
```

---

## ✅ 6.3 Add Exposure Modifier

If exposure == public:

```
Risk × 1.3
```

---

## ✅ 6.4 Map to Priority

Example:

```
Score >= 5 → P1
Score >= 4 → P2
Score >= 3 → P3
Else → P4
```

---

# 📜 7️⃣ Policy Engine

## ✅ 7.1 Evaluate Policy Rules

Example rule:

Block if:

* severity = CRITICAL
* verdict = CONFIRMED

Return:

```json
{
  "pipeline_status": "BLOCK"
}
```

---

# 📊 8️⃣ Final Report Builder

## ✅ 8.1 Build Unified JSON

Structure:

```json
{
  "scan_id": "...",
  "summary": {
    "total_vulns": 20,
    "confirmed": 4,
    "likely": 6,
    "not_observed": 10
  },
  "findings": [...]
}
```

---

## ✅ 8.2 Add Executive Summary (Rule-Based Only)

No LLM yet.

Example:

> 4 confirmed reachable vulnerabilities detected in publicly exposed service.

---

# 🧪 9️⃣ Testing Strategy

## ✅ 9.1 Unit Tests

* Config parser
* Trivy output parser
* Correlation logic
* Risk scoring
* Policy evaluation

---

## ✅ 9.2 Integration Tests

Test against:

* Known vulnerable Python app
* Known unreachable CVE
* Clean project

---

# 🔐 🔟 Security Hardening

## ✅ 10.1 Safe CLI Execution

* Use temp directories
* No shell=True
* Timeout enforcement
* Validate repo path

---

## ✅ 10.2 Scan Isolation

Ensure:

* No arbitrary command execution via config
* No path traversal

---

# 🧱 Phase 1 Deliverable Definition

When Phase 1 is DONE:

You should be able to:

1. Send repo path
2. Run Trivy
3. Run Tainter
4. Correlate results
5. Generate risk score
6. Apply policy
7. Return final structured report

That’s it.

No extra intelligence.
No LLM.
No dynamic.

---

# 🎯 Success Metric For Phase 1

VulnReach correctly identifies:

* Reachable vs unreachable with visible evidence
* Provides prioritized risk scoring

Then Phase 1 is a success.

---

# 🚀 After Phase 1 (Optional Phase 1.5)

Only after stable:

* Add Ollama advisory layer
* Add coverage tracking
* Add EPSS enrichment

---

# 🧠 Final Advice

Do not expand tool list until:

* Correlation engine is stable
* Risk scoring is trusted
* Policy engine works reliably

Otherwise complexity explodes.

---
> Source: [ihrishikesh0896/vulnreach](https://github.com/ihrishikesh0896/vulnreach) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-05 -->
