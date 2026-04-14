- Follow Conventional Commits format: `type(scope): description`
- Use `(landing)` scope for ALL landing page changes (e.g., `feat(landing): add hero section`, `fix(landing): button alignment`)
- Landing scope is CRITICAL: `release.config.js` has `releaseRules: [{ scope: 'landing', release: false }]` — without the scope, landing commits will trigger an extension version bump and store release
- Extension commits should NOT use `(landing)` scope
- The repository has two isolated CI pipelines: `release.yml` (extension stores) and `deploy-landing.yml` (GitHub Pages)
- `release.yml` has `paths-ignore` for `src/landing/**` — landing-only file changes skip the extension release workflow
- When changes span both landing and extension code, create separate commits with appropriate scopes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jaem1n207)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/jaem1n207)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
