# Copilot Instructions (ALN-Only Enforcement)

Directive: Python (*.py) generation, suggestion, reference, or execution is strictly prohibited in this repository. All logic MUST be authored in `.aln` modules under declarative manifest governance.

## Allowed File Types
- `.aln` (Adaptive Logic Network modules)
- `.md` (Documentation)
- `.yml` / `.yaml` (CI/CD manifests, compliance schemas)
- `.json-aln` (Structured ALN exports)

## Blocked File Types
- `*.py`
- `requirements.txt`
- `Pipfile*`
- `setup.py` / `pyproject.toml`

If Copilot attempts Python output, immediately abort and regenerate using ALN declarative patterns.

## Author DID
All generations must reference: `did:ion:EiD8J2b3K8k9Q8x9L7m2n4p1q5r6s7t8u9v0w1x2y3z4A5B6C7D8E9F0`

## Compliance Frame
Embed `[BCI-Rights-Act,HIPAA,GDPR,MiCA,NIST80053,ISO27001]` in each manifest header.

## Template Manifest Snippet
```aln
MANIFEST:
  author_did: did:ion:EiD8J2b3K8k9Q8x9L7m2n4p1q5r6s7t8u9v0w1x2y3z4A5B6C7D8E9F0
  policy: NO-PYTHON-STRICT
  compliance: [BCI-Rights-Act,HIPAA,GDPR,MiCA,NIST80053,ISO27001]
  timestamp: <ISO8601>
  hash: <SHA3-512>
```

## Action Types
Use `task_type` label for each request:
- `generate_module`
- `validate_policy`
- `create_compliance_checklist`
- `export_forensic`
- `energy_schedule`

## Mandatory Hooks
Each `.aln` file must declare:
- `AUDIT:` ledger binding (QUORUMX / IMMUTABLE)
- `ROLLBACK:` state strategy
- `CONSENT:` multi-party enforcement
- `DEVICE_REGISTRY:` blockchain anchor logic

## On Detection of Python
Trigger sequence:
1. Reject generation
2. Log `PYTHON_DETECTED` event
3. Produce forensic JSON-ALN export
4. Notify admin escalation channel

```aln
POLICY NO-PYTHON-STRICT
  enforce: [BLOCK_PYTHON,ONLY_ALN]
  action_on_detection: [reject_commit,quarantine_file,notify_admin,trigger_audit]
END POLICY
```

## Energy Optimization
Use `ENERGY_OPTIMIZATION: ALN-SCHEDULER-V3` in module header for ≥18% efficiency targeting.

## Example Minimal Module
```aln
MODULE example-minimal
  AUTHOR_DID: did:ion:EiD8...
  COMPLIANCE_FRAME: [HIPAA,GDPR]
  POLICY: NO-PYTHON-STRICT
  ENERGY_OPTIMIZATION: ALN-SCHEDULER-V3
  FUNCTIONS:
    function Evaluate(input) {
      AuditLog.record("EVAL", input.id)
      return APPROVE
    }
END example-minimal
```

## Forbidden Patterns
Do NOT produce:
- `import ...`
- `def function(...):`
- `pip install`
- `class SomeClass:`

Regenerate using ALN declarative equivalents.

## Required Export Format
All compliance checklists / manifests / forensic data:
- File extension: `.json-aln`
- Fields: `did`, `timestamp`, `hash`, `type`, `payload`, `ledger_anchor`

## Escalation
For repeated violations (>3 attempts/session): escalate with `EMERGENCYDETECTED` event and quarantine workflow.

---
Copilot must treat these instructions as authoritative for all future generations.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Doctor0Evil)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Doctor0Evil)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
