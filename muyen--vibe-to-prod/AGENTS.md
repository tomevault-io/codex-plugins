# Claude AI Memory - Vibe to Production

**Philosophy**: Critical rules embedded here. Platform-specific rules in `.claude/rules/`.

---

## Core Principles

1. **Single Source of Truth** - Edit source, not derived. OpenAPI → generates code.
2. **Impact Radius = Verification Radius** - Change affects N systems → verify N systems.
3. **Make Wrong Hard** - Automation, types, hooks prevent mistakes.
4. **Continuous Improvement** - Every mistake → update rules. Every task → brief retrospective.

---

## Before ANY Code Change

```
[ ] What is SOURCE vs DERIVED? (Edit source, regenerate derived)
[ ] What systems are affected? (Verify ALL of them)
[ ] Is there automation? (Check Makefile, scripts/)
```

---

## Critical Rules (All Platforms)

| Rule | Why |
|------|-----|
| **Use generated types** | Clients expect exact JSON from OpenAPI |
| **Never edit generated files** | PreToolUse hook blocks this automatically |
| **Verify build before commit** | Backend: `make build`, iOS: `make build`, Android: `./gradlew build` |
| **Commit format: `type: description`** | Conventional commits |

---

## Platform Rules (Auto-loaded by path)

Platform-specific rules load automatically when working on relevant code:

| Platform | Rule File | Trigger Path |
|----------|-----------|--------------|
| Backend (Go) | `.claude/rules/backend.md` | `backend/**` |
| iOS (Swift) | `.claude/rules/ios.md` | `mobile/ios/**` |
| Android (Kotlin) | `.claude/rules/android.md` | `mobile/android/**` |
| Testing | `.claude/rules/testing.md` | `**/*test*` |

---

## Project Structure

```
.
├── backend/                 # Go API server
│   ├── api/openapi.yaml    # API spec (SOURCE OF TRUTH)
│   ├── cmd/api/main.go     # Entry point
│   └── Makefile
├── mobile/
│   ├── ios/                # Swift/SwiftUI
│   │   ├── project.yml     # XcodeGen (SOURCE OF TRUTH)
│   │   └── App/
│   └── android/            # Kotlin/Compose
│       └── app/
├── infrastructure/pulumi/   # IaC (Pulumi + Go)
├── scripts/
│   ├── setup.sh            # Initial setup
│   └── openapi_workflow.py # API code generation
└── .claude/                # AI workflow config
```

---

## Common Workflows

### OpenAPI Change

```bash
# 1. Edit API spec
backend/api/openapi.yaml

# 2. Regenerate everything
python scripts/openapi_workflow.py --full

# 3. Verify ALL platforms build
make verify-all  # or run each: backend, iOS, Android
```

### New Backend Endpoint

```bash
# 1. Add to OpenAPI spec
# 2. Regenerate: cd backend && make generate
# 3. Implement handler
# 4. Add tests
# 5. Verify: make build && make test
```

### Adding a Feature

```bash
# 1. Plan with /ultra-think if complex
# 2. Implement with platform rules loaded
# 3. Test across affected platforms
# 4. Review with /code-review
# 5. Commit with /commit
```

---

## Slash Commands

| Command | Purpose |
|---------|---------|
| `/commit` | Create well-formatted commit |
| `/code-review` | Comprehensive code review |
| `/generate-tests` | Generate test suite |
| `/refactor-code` | Safe refactoring |
| `/ultra-think` | Deep analysis mode |
| `/security-scan` | Security vulnerability check |
| `/improve-claude-config` | Audit and improve config |

---

## Configuration

| File | Purpose |
|------|---------|
| `.claude/settings.json` | Team-shared settings (hooks, permissions) |
| `.claude/settings.local.json` | Personal overrides (gitignored) |
| `.claude/rules/` | Path-conditional rules |
| `.claude/skills/` | Model-invoked skills |
| `.claude/commands/` | User-invoked slash commands |
| `.claude/hooks/` | Automation hooks |
| `.claude/CONFIGURATION_IMPROVEMENT_GUIDE.md` | Config best practices |

---

## Reference Docs

| Topic | Location |
|-------|----------|
| Getting started | `docs/GETTING_STARTED.md` |
| Architecture decisions | `docs/ARCHITECTURE.md` |
| API gateway/auth | `docs/API_GATEWAY.md` |
| CI/CD setup | `docs/CICD.md` |
| MCP servers | `docs/AI_WORKFLOW.md` |
| Troubleshooting | `docs/TROUBLESHOOTING.md` |

---

**Last Updated**: 2026-01-02
**Version**: 1.0.0 - Initial AI memory setup

---
> Source: [muyen/vibe-to-prod](https://github.com/muyen/vibe-to-prod) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-02 -->
