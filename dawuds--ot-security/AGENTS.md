# OT-Security — IEC 62443 + NACSA OT Security

**Last updated:** 2026-03-31

## What This Is

**Tier 3 Supporting Framework**

Structured knowledge base for OT/ICS/SCADA security based on IEC 62443 and NACSA sector requirements. SPA explorer with JSON data layers.

## Portfolio Role

Tier 3 — Supporting Framework. IEC 62443 and NACSA OT compliance reference for OT/ICS/SCADA environments. Supports Tier 1 repos (NACSA for NCII-designated OT sectors, IESP for OT assessments).

## Quick Start

Open `index.html` in a browser. Run `node validate.js` to check data integrity.

## Architecture
- **SPA**: `index.html` + `app.js` + `style.css` (vanilla JS, no build step)
- **Data**: JSON files across controls, cross-references, sectors, standards, risk-management, evidence, requirements
- **Schema**: GRC Portfolio v2.0 Standardized Schema

## Key Data Files
- `controls/library.json` — 25 controls focused on ICS/SCADA/OT environments
- `controls/domains.json` — 13 domains
- `sectors/requirements/` — 6 sector profiles (energy, transport, water, manufacturing, oil-gas, building-automation)
- `sectors/purdue-model.json` — Purdue Enterprise Reference Architecture mapping
- `standards/iec62443/`, `standards/nist-800-82/`, `standards/mitre-attack-ics/`

- `cross-references/iec62443-to-nacsa.json` — IEC 62443 to NACSA CoP mapping
- `cross-references/iec62443-to-nist-csf.json` — IEC 62443 to NIST CSF
- `cross-references/iec62443-to-nist80082.json` — IEC 62443 to NIST 800-82
- `cross-references/mitre-to-controls.json` — MITRE ATT&CK for ICS mapping
- `cross-references/sector-to-nacsa-cop.json` — Sector to NACSA CoP alignment
- `audit-integration.json` — Maps 25 controls to Tech-Audit/OT-Security audit procedures

## Conventions
- Kebab-case slugs for all IDs
- Security Levels (SL 1-4) per IEC 62443 zones and conduits
- Sector-specific requirements extend base controls

## Important
- OT environments have safety implications — never weaken controls without safety review
- Purdue model levels must be preserved in network segmentation controls
- MITRE ATT&CK for ICS tactics are OT-specific, not enterprise ATT&CK

## Validation

```bash
node validate.js
```

## Related Repos
- [nacsa](https://github.com/dawud-shakeel/nacsa) — NACSA Act 854 (Tier 1 Core Framework — OT sectors: Energy, Transport, Water are NCII sectors)
- [iesp](https://github.com/dawud-shakeel/iesp) — IESP OT assessments (Tier 1 Core Framework)
- [Tech-Audit](https://github.com/dawud-shakeel/Tech-Audit) — Technology Risk Audit including OT sector guides (Tier 2 Operational)
- [RMIT](https://github.com/dawud-shakeel/RMIT) — BNM RMiT compliance (Tier 3 Supporting Framework)
- [grc](https://github.com/dawud-shakeel/grc) — GRC Portfolio Hub

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dawuds)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/dawuds)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
