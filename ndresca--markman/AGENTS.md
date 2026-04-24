# gstack

Use the `/browse` skill from gstack for all web browsing. Never use `mcp__claude-in-chrome__*` tools directly.

If gstack skills aren't working, run `cd .claude/skills/gstack && ./setup` to build the binary and register skills.

## Available gstack skills

- `/office-hours` — YC-style office hours for startup/project brainstorming
- `/plan-ceo-review` — CEO/founder-mode plan review
- `/plan-eng-review` — Eng manager-mode architecture review
- `/plan-design-review` — Designer's eye plan review
- `/design-consultation` — Full design system consultation
- `/review` — Pre-landing PR code review
- `/ship` — Ship workflow: tests, changelog, PR creation
- `/land-and-deploy` — Merge PR, wait for CI/deploy, verify production
- `/canary` — Post-deploy canary monitoring
- `/benchmark` — Performance regression detection
- `/browse` — Fast headless browser for QA and site testing
- `/qa` — Systematically QA test and fix bugs
- `/qa-only` — QA report only (no fixes)
- `/design-review` — Designer's eye visual QA with fixes
- `/setup-browser-cookies` — Import cookies from real browser for authenticated testing
- `/setup-deploy` — Configure deployment settings
- `/retro` — Weekly engineering retrospective
- `/investigate` — Systematic root cause debugging
- `/document-release` — Post-ship documentation update
- `/codex` — OpenAI Codex CLI code review / challenge / consult
- `/cso` — Chief Security Officer security audit
- `/careful` — Safety guardrails for destructive commands
- `/freeze` — Restrict edits to a specific directory
- `/guard` — Full safety mode (careful + freeze combined)
- `/unfreeze` — Clear freeze boundary
- `/gstack-upgrade` — Upgrade gstack to latest version

## Design System
Always read DESIGN.md before making any visual or UI decisions.
All font choices, colors, spacing, and aesthetic direction are defined there.
Do not deviate without explicit user approval.
In QA mode, flag any code that doesn't match DESIGN.md.

Key tokens at a glance:
- Fonts: EB Garamond (logo/display/headers), Instrument Sans (UI/body), Geist Mono (data/numbers)
- Colors: Navy #0A1628 (text), Blue #2563EB (CTAs only), White #FFFFFF (bg), #FAFAFA (surface), #E5E7EB (border)
- Semantic: Success #16A34A, Warning #D97706, Danger #DC2626
- Radius: sm=4px, md=6px, lg=8px

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ndresca) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
