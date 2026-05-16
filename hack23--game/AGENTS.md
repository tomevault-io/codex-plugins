# Copilot Instructions

## Project Overview

**Target Shooter** — a 3D target-clicking game built with React 19, TypeScript 6, Three.js (`@react-three/fiber` + `@react-three/drei`), Vite 8, and Howler.js. **Node ≥ 25** required.

**Security & Compliance:** Aligned with [Hack23 AB's ISMS](https://github.com/Hack23/ISMS-PUBLIC). Every change MUST respect the ISMS. See [ISMS Policy Mapping](../docs/ISMS_POLICY_MAPPING.md) for the full feature-to-policy traceability matrix.

## Required Context

Before starting any non-trivial task, read:

1. `.github/workflows/copilot-setup-steps.yml` — Copilot environment & permissions
2. `.github/copilot-mcp.json` — MCP server wiring (GitHub Insiders, filesystem, memory, sequential-thinking, playwright)
3. `README.md` — project overview, features, security posture
4. `.github/agents/README.md` & `.github/skills/README.md` — available specialists and patterns
5. The specific agent and skill files relevant to your task
6. [Hack23 ISMS-PUBLIC](https://github.com/Hack23/ISMS-PUBLIC) — authoritative security policies

## Quick Reference

```bash
npm install           # Install dependencies
npm run dev           # Dev server at http://localhost:5173
npm run build         # TypeScript check + Vite build
npm run lint          # ESLint
npm run test          # Vitest unit tests
npm run coverage      # Tests with coverage (≥80% lines, ≥70% branches)
npm run test:e2e      # Cypress E2E tests
npm run test:licenses # License compliance check (Open Source Policy)
npm audit             # Dependency vulnerability scan
```

## Project Structure

```text
src/
├── App.tsx                    # Root: game loop, audio, overlays
├── components/
│   ├── GameScene.tsx          # 3D scene (targets, particles, lights)
│   ├── TargetSphere.tsx       # Clickable 3D target
│   ├── BackgroundParticles.tsx # Particle effects
│   ├── ParticleExplosion.tsx   # Hit explosion effect
│   ├── HUD.tsx                # Score/time/combo display
│   └── GameOverlay.tsx        # Controls, instructions, overlays
├── hooks/
│   ├── useGameState.ts        # Game state management
│   └── useAudioManager.ts     # Howler.js audio integration
├── utils/
│   ├── gameConfig.ts          # Game constants and level progression
│   └── targetPhysics.ts       # Target creation and movement
└── test/setup.ts              # Vitest test setup
```

## ISMS Policy Quick Map

| Applies When… | Policy | Link |
|---|---|---|
| Any security-relevant code, governance, incident | Information Security Policy | [ISP](https://github.com/Hack23/ISMS-PUBLIC/blob/main/Information_Security_Policy.md) |
| SDLC, CI/CD, testing, deployment, threat modeling | Secure Development Policy | [SDP](https://github.com/Hack23/ISMS-PUBLIC/blob/main/Secure_Development_Policy.md) |
| Adding / updating / removing dependencies, licenses, SBOM | Open Source Policy | [OSP](https://github.com/Hack23/ISMS-PUBLIC/blob/main/Open_Source_Policy.md) |
| Copilot agents, MCP, LLM-assisted changes | AI Policy | [AI](https://github.com/Hack23/ISMS-PUBLIC/blob/main/AI_Policy.md) |
| Auth, identity, permissions, least privilege | Access Control Policy | [ACP](https://github.com/Hack23/ISMS-PUBLIC/blob/main/Access_Control_Policy.md) |
| Encryption, hashing, key material, TLS | Cryptography Policy | [CP](https://github.com/Hack23/ISMS-PUBLIC/blob/main/Cryptography_Policy.md) |
| Data handling, classification, storage, logs | Data Classification Policy | [DCP](https://github.com/Hack23/ISMS-PUBLIC/blob/main/Data_Classification_Policy.md) |
| Personal data, analytics, telemetry | Privacy Policy | [Privacy](https://github.com/Hack23/ISMS-PUBLIC/blob/main/Privacy_Policy.md) |
| Changes to agents, MCP, workflows | Change Management | [CM](https://github.com/Hack23/ISMS-PUBLIC/blob/main/Change_Management.md) |
| STRIDE, abuse cases, attack surface | Threat Modeling Policy | [TM](https://github.com/Hack23/ISMS-PUBLIC/blob/main/Threat_Modeling.md) |
| Reporting / patching vulnerabilities | Vulnerability Management | [VM](https://github.com/Hack23/ISMS-PUBLIC/blob/main/Vulnerability_Management.md) |

Cite the applicable policy in code comments, commit messages, and PR descriptions for security-sensitive changes.

## Secure SDLC Phases (per SDP)

1. **Plan & Design** — classify impact (CIA triad), threat model (STRIDE), link to ISMS policies
2. **Develop** — OWASP-aligned coding, no hardcoded secrets, typed everywhere, `no-any`
3. **Test** — SAST (CodeQL + ESLint), SCA (`npm audit`), DAST (ZAP), unit + E2E, coverage ≥80%/70%
4. **Deploy** — CI/CD gates, SHA-pinned Actions, SLSA ≥ L3 attestations, SBOM + SBOMQS ≥ 7.0
5. **Operate** — monitor CodeQL alerts, Dependabot, Scorecard ≥ 8.0, patch per Vulnerability Management SLAs

## AI-Augmented Development Controls (ISMS SDP §AI-Augmented Controls)

1. **AI outputs are proposals, never authoritative** — human review required before merge
2. **No autonomous deployment** — AI may not bypass CI gates, branch protections, or approvals
3. **Change attribution** — document AI assistance in PR descriptions (e.g., "assisted by Copilot agent `game-developer`")
4. **Least privilege tooling** — each agent has tools appropriate to its role; expansion is a Change Management event
5. **Curator-agent changes** — edits to `.github/agents/*.md`, `.github/skills/*/SKILL.md`, `.github/copilot-mcp.json`, `.github/workflows/copilot-setup-steps.yml` are Normal Changes that require CEO or security-owner approval
6. **Audit trail** — all agent activity logged via GitHub; PRs preserve complete traceability
7. **MCP governance** — MCP server config (`.github/copilot-mcp.json`) uses `secrets.COPILOT_MCP_GITHUB_PERSONAL_ACCESS_TOKEN`, never hard-coded tokens

## Coding Standards

- **TypeScript strict**: `strictNullChecks`, `noImplicitAny`, `noUncheckedIndexedAccess` — do not disable
- **No `any`**: prefer `unknown` + narrowing; type every function signature
- **Functional components only** with explicit return types: `function X(): JSX.Element`
- **Props interfaces**: always define, JSDoc-documented
- **Import order**: React → external libs → internal components → hooks → types → styles
- **No secrets in code, logs, or errors** — use environment / `secrets.*` only
- **Validate every external input** — client *and* server — per Security-by-Design skill

## Three.js Patterns

- Declarative JSX (`@react-three/fiber`), not imperative Three.js API
- `useFrame((state, delta) => …)` for animation — never `setInterval`/`setTimeout`
- Type refs: `useRef<THREE.Mesh>(null)` — never untyped
- Mesh event props (`onClick`, `onPointerOver`) — not DOM listeners
- Dispose geometries/materials/textures on unmount
- `InstancedMesh` for > 10 similar objects
- Target **60 fps** (≤ 16.67 ms per frame)

## Testing (per SDP §Unit Test Coverage & Quality)

- **Vitest + jsdom** for unit tests, **Cypress** for E2E
- Colocate tests: `ComponentName.test.tsx` next to source
- **≥ 80% line, ≥ 70% branch coverage** on every module
- **≥ 95% coverage** on security-sensitive code (input validation, auth, encoding)
- AAA pattern (Arrange-Act-Assert), deterministic (mock `Date.now`, `Math.random`, timers)
- Mock `@react-three/fiber` and `@react-three/drei` in unit tests
- Test behavior, not implementation

## Dependencies & Supply Chain (per OSP)

- **Approved licenses**: MIT, Apache-2.0, BSD-2/3-Clause, ISC, CC0-1.0, Unlicense
- **Review required**: LGPL 2.1/3.0, MPL 2.0, EPL 2.0 (weak copyleft)
- **Prohibited without CEO approval**: GPL 2.0/3.0, AGPL 3.0, advertising-clause, incompatible licenses
- Before adding a dependency: `npm audit` + `npm run test:licenses` + vulnerability check
- All GitHub Actions **pinned to full commit SHA**, never tags/branches
- SBOM generated per release with SBOMQS ≥ 7.0 and SLSA L3 attestation

## Rules

### ALWAYS
1. Use TypeScript strict mode with explicit types
2. Run `npm run lint` and `npm run test` before committing
3. Maintain ≥ 80% line / ≥ 70% branch coverage (≥ 95% on security code)
4. Use `useFrame` with delta time for animations
5. Dispose Three.js resources in cleanup functions
6. Validate + sanitize all external input (see security-by-design skill)
7. Cite ISMS policy in comments/PRs for security changes
8. Document AI-assistance in PR descriptions when using Copilot agents

### NEVER
1. Use `any` — use `unknown` with narrowing
2. Commit secrets, API keys, credentials, PII, or production data (including test data)
3. Use `setInterval`/`setTimeout` for game animations
4. Update state inside `useFrame` (use refs for high-frequency updates)
5. Add dependencies without `npm audit` + license check
6. Use GPL/AGPL licensed dependencies without CEO approval
7. Pin Actions to tags or branches — always full SHA
8. Bypass CI security gates or branch protections (even with AI assistance)

## Naming Conventions

- Components: PascalCase (`TargetSphere`)
- Hooks: `use` prefix (`useGameState`)
- Types/Interfaces: PascalCase (`GameState`, `Target`)
- Constants: UPPER_SNAKE_CASE (`GAME_DURATION`)
- Functions: camelCase (`incrementScore`)

## Decision Framework

- **Complete without asking** when: pattern exists in codebase, requirements are clear, change is small and isolated, ISMS impact is understood
- **Ask first** when: multiple approaches with security/performance tradeoffs, new dependencies, new MCP/agent capability, changes to CI/security gates, potential privacy impact

---
> Source: [Hack23/game](https://github.com/Hack23/game) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-15 -->
