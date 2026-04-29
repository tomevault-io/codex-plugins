# Code Review Instructions

## Project Context

**Stack:**

- Language: Nix (NixOS configuration), YAML (Home Assistant), Jinja2 (templates)
- Architecture: Declarative GitOps with immutable infrastructure
- Build System: Nix Flakes with locked dependencies
- Key Frameworks: NixOS 24.11, Home Assistant, Wyoming Protocol (Whisper STT, Piper TTS)

**Core Modules:**

- `hosts/homelab/`: System configuration (boot, networking, SSH, Comin)
- `hosts/homelab/home-assistant/`: HA service, voice processing, intents, automations
- `tests/`: Configuration validation (automation IDs, entities, services, YAML schema)
- `custom_sentences/pl/`: Polish voice command sentence patterns

**Conventions:**

- Automation IDs: `snake_case` (e.g., `startup_notification`)
- Automation aliases: `Category - Description` (e.g., `System - Startup notification`)
- Entity names: Polish lowercase, spaces → underscores (e.g., `light.salon`)
- Intent names: PascalCase (e.g., `TurnOnLight`)
- Entity ID normalization: `{{ slots.name | lower | replace(' ', '_') }}`
- Formatting: `alejandra` for Nix, 2-space indent for YAML
- Deployment: Git commit → Comin auto-pull (~60s) → nixos-rebuild
- Documentation: Inline for complex Nix expressions, Polish voice patterns

**Critical Areas (Extra Scrutiny):**

- Voice command intent matching (Polish language patterns)
- Automation logic (service calls, entity IDs, domain compatibility)
- GitOps integrity (immutable config, no config drift)
- Secrets handling (must NOT commit .secret, .key, secrets.yaml)
- Home Assistant configuration validation (test coverage required)

---

## Review Before CI Completes

You review PRs immediately, before CI finishes. Do NOT flag issues that CI will catch.

**CI Already Checks:**

- Nix code formatting (`alejandra`)
- YAML syntax and linting (`yamllint`)
- Markdown linting (`markdownlint-cli2`)
- GitHub Actions workflow linting (`actionlint`)
- Configuration validation (`nix flake check`)
- Schema validation (intent structure, Jinja2, HA services)

---

## Review Priority Levels

### 🔴 CRITICAL (Must Block PR)

**Security Vulnerabilities** (95%+ confidence)

- [ ] Secrets in code (SSH keys, API tokens, passwords in .nix files)
- [ ] Secrets files not in .gitignore (.secret, .key, secrets.yaml)
- [ ] SSH configuration allows password auth or root login
- [ ] Home Assistant exposed without authentication
- [ ] Hardcoded credentials in voice intents or automations
- [ ] Sensitive data in logs or automation descriptions

**Correctness Issues** (90%+ confidence)

- [ ] Invalid entity IDs (wrong domain.name format)
- [ ] Service calls to non-existent services
- [ ] Domain mismatch (e.g., `light.turn_on` with `switch.` entity)
- [ ] Duplicate automation IDs (causes HA conflicts)
- [ ] Breaking changes to voice intents without updating custom_sentences
- [ ] Unbalanced Jinja2 delimiters ({{ }}, {% %})
- [ ] Invalid YAML structure in custom_sentences/pl/
- [ ] GitOps drift (manual changes not in Git)
- [ ] Flake.lock not updated after input changes

### 🟡 HIGH (Request Changes)

**Maintainability** (80%+ confidence)

- [ ] Voice intents without corresponding custom_sentences patterns
- [ ] New automations without test coverage in config-validation.nix
- [ ] Complex Nix expressions without comments
- [ ] Polish voice patterns missing verb alternatives or skip words
- [ ] Service actions without state feedback (silent failures)
- [ ] Entity names not following Polish lowercase convention
- [ ] Automation aliases not following "Category - Description" format
- [ ] New Home Assistant modules without documentation

**Architecture** (75%+ confidence)

- [ ] GUI-based Home Assistant config (should be declarative in .nix)
- [ ] Hardcoded entity IDs (should use input helpers or templates)
- [ ] Duplicated automation logic (should extract to scripts)
- [ ] Direct service calls instead of intent scripts
- [ ] Missing Polish entity name normalization filter
- [ ] Tight coupling between intents and specific devices
- [ ] Over-engineering simple automations

### 🟢 MEDIUM (Suggest/Comment)

**Performance** (70%+ confidence)

- [ ] Heavy Whisper STT model without justification (current: small)
- [ ] Unnecessary automation triggers (too frequent polling)
- [ ] Large custom_sentences patterns (increases intent processing time)
- [ ] Blocking operations in automation scripts

**Best Practices** (65%+ confidence)

- [ ] Missing error handling in voice intent responses
- [ ] No TTS feedback for failed actions
- [ ] Entity IDs not descriptive enough
- [ ] Automations without conditions (always execute)
- [ ] Missing input validation in voice slot values

### ⚪ LOW (Optional/Skip)

Don't comment on:

- Personal naming style (if follows conventions)
- Minor optimizations with no measurable impact
- Alternative Nix expression styles (if valid)
- Refactoring unrelated to the change
- Anything below confidence threshold

---

## Security Deep Dive

### Secrets Management

- [ ] NO secrets in .nix, .yaml, or .md files
- [ ] Secrets referenced via environment variables or external files
- [ ] .gitignore includes: `*.secret`, `*.key`, `secrets.yaml`, `.env`
- [ ] Consider sops-nix or agenix for encrypted secrets in Git
- [ ] SSH keys managed via `openssh.authorizedKeys.keys` (public only)

### Authentication

- [ ] SSH password authentication disabled (`PasswordAuthentication no`)
- [ ] Root login disabled (`PermitRootLogin no`)
- [ ] Home Assistant requires login on first access
- [ ] No default credentials in configuration

### Input Validation

- [ ] Voice command slot values validated (e.g., brightness 0-100)
- [ ] Entity IDs sanitized: `{{ slots.name | lower | replace(' ', '_') }}`
- [ ] Service domain matches entity domain (enforced in tests)
- [ ] YAML schema validated in tests/schema-validation.nix

### Data Protection

- [ ] Home Assistant state in `/var/lib/hass/` (not in Git)
- [ ] No PII in automation descriptions or logs
- [ ] HTTPS enforced if exposed to internet (currently local-only)

---

## Code Quality Standards

### Naming

- Nix attributes: `camelCase` or `snake_case` (NixOS convention)
- Automation IDs: `snake_case` (e.g., `light_control_bedroom`)
- Automation aliases: `Category - Description` (e.g., `Lights - Bedroom control`)
- Entity IDs: `domain.polish_name` (e.g., `light.sypialnia`)
- Intent names: `PascalCase` (e.g., `SetBrightness`)
- Meaningful names (intent clear without comments)

### Error Handling

- All voice intents return TTS feedback (success or failure)
- Service calls wrapped in conditionals checking entity state
- Automation conditions prevent invalid states
- Test coverage validates all service/entity combinations

### Testing

- **Coverage:** 100% for automations with unique IDs
- **Required tests:**
  - [ ] New automations have unique IDs (config-validation.nix)
  - [ ] New service calls validated (domain.action format)
  - [ ] New entity IDs validated (domain.name format)
  - [ ] Domain compatibility tested (service domain matches entity domain)
  - [ ] YAML syntax validated (schema-validation.nix)
  - [ ] Jinja2 templates balanced ({{ }}, {% %})
- **Test execution:** `nix flake check` must pass before merge

### Documentation

- [ ] Complex Nix expressions explained
- [ ] Polish voice patterns documented (intent → sentence mapping)
- [ ] New voice commands added to README examples
- [ ] Breaking changes noted in PR description
- [ ] CLAUDE.md updated if new patterns emerge

### Configuration Integrity

- [ ] All changes in Git (no manual edits on server)
- [ ] Flake.lock updated if dependencies changed
- [ ] `nix fmt` applied before commit
- [ ] No config drift (declarative source of truth)

---

## Language-Specific Guidelines

### Nix

- Use `lib.mkIf` for conditional config, not if-then-else
- Prefer `let...in` for complex expressions
- Import modularly: separate intents, automations, helpers
- Use `mkDefault` for overridable values
- Document non-obvious attribute merging
- Format with `alejandra` before commit
- NEVER use `eval` or `import from derivation` (IFD) without justification

### YAML (Home Assistant Custom Sentences)

- 2-space indentation (enforced by yamllint)
- Max 120 chars per line
- Use `lists` for sentence alternatives
- Include `expansion_rules` for skip words
- All intents must have `data` section with example responses
- Polish diacritics required (ą, ć, ę, ł, ń, ó, ś, ź, ż)

### Jinja2 (Templates)

- Always balance delimiters: `{{ }}` for expressions, `{% %}` for statements
- Use filters for entity ID normalization: `| lower | replace(' ', '_')`
- Avoid complex logic (move to scripts if >3 filters)
- Test template rendering with example values
- Escape user input in TTS responses

### Home Assistant Configuration

- Prefer declarative .nix config over GUI
- Use `input_boolean`, `input_select` for dynamic state
- Scripts for reusable action sequences
- Templates for dynamic entity references
- Automation IDs mandatory and unique
- Conditions before actions to prevent invalid states

---

## Architecture Patterns

**Follow these patterns:**

- Declarative GitOps: all config in Git, Comin auto-deploys
- Immutable infrastructure: nixos-rebuild replaces entire system state
- Modular composition: separate files for intents, automations, helpers
- Voice-first design: all actions accessible via Polish voice commands
- Entity ID normalization: consistent lowercase underscore format

**Avoid these anti-patterns:**

- GUI-based Home Assistant configuration (creates config drift)
- Hardcoded entity IDs in automations (use templates or input helpers)
- Manual server edits (overwritten by nixos-rebuild)
- Skipping tests (`nix flake check` must pass)
- Secrets in Git (use sops-nix or agenix)
- Over-engineering (simple automations stay simple)

---

## Review Examples

### ✅ Good: Parameterized Entity ID

```nix
service = "light.turn_on";
target.entity_id = "light.{{ slots.name | lower | replace(' ', '_') }}";
```

### ❌ Bad: Hardcoded Entity ID

```nix
service = "light.turn_on";
target.entity_id = "light.salon";  # Inflexible, breaks voice intent
```

---

### ✅ Good: Unique Automation ID

```nix
automation = [
  {
    id = "startup_notification";  # Unique, descriptive
    alias = "System - Startup notification";
    # ...
  }
];
```

### ❌ Bad: Missing or Duplicate ID

```nix
automation = [
  {
    # Missing ID - causes HA conflicts
    alias = "Startup notification";
    # ...
  }
];
```

---

### ✅ Good: Domain Compatibility

```nix
service = "light.turn_on";
target.entity_id = "light.sypialnia";  # Domains match
```

### ❌ Bad: Domain Mismatch

```nix
service = "light.turn_on";
target.entity_id = "switch.sypialnia";  # Service domain != entity domain
```

---

### ✅ Good: Voice Feedback

```nix
then = [
  {
    service = "light.turn_on";
    target.entity_id = "light.{{ slots.name | lower | replace(' ', '_') }}";
  }
  {
    service = "tts.speak";
    data.message = "Włączam światło {{ slots.name }}";
  }
];
```

### ❌ Bad: Silent Failure

```nix
then = [
  {
    service = "light.turn_on";
    target.entity_id = "light.{{ slots.name | lower | replace(' ', '_') }}";
  }
  # No TTS feedback - user doesn't know if it worked
];
```

---

### ✅ Good: Modular Nix Imports

```nix
# hosts/homelab/home-assistant/default.nix
imports = [
  ./intents.nix
  ./automations.nix
];
```

### ❌ Bad: Monolithic Configuration

```nix
# Everything in one file - hard to maintain
services.home-assistant = {
  # 500 lines of intents + automations + helpers...
};
```

---

### ✅ Good: Test Coverage for New Automation

```nix
# tests/config-validation.nix
uniqueAutomationIds = let
  ids = map (a: a.id) config.automations;
  duplicates = lib.filter (id: (lib.count (x: x == id) ids) > 1) ids;
in
  if duplicates == []
  then null
  else throw "Duplicate automation IDs: ${lib.concatStringsSep ", " duplicates}";
```

---

## Maintainer Priorities

**What matters most to this project:**

1. **Correctness:** Voice commands must work reliably (Polish patterns, entity matching)
2. **GitOps discipline:** All config in Git, no manual server edits, immutable deployments
3. **Test coverage:** Configuration validation catches errors before deployment
4. **Simplicity:** Small codebase (~1300 LOC), straightforward automations, no over-engineering

**Trade-offs we accept:**

- Configuration verbosity for clarity (declarative Nix over clever abstractions)
- Polish-only voice support (no multilingual complexity)
- Local-only access (no remote security overhead yet)
- Conservative error handling over aggressive optimization

---

## Confidence Threshold

Only flag issues you're **80% or more confident** about.

If uncertain:

- Phrase as question: "Could this cause voice intent matching to fail?"
- Suggest investigation: "Consider testing this entity ID format with Polish diacritics"
- Don't block PR on speculation

---

## Review Tone

- **Constructive:** Explain WHY, not just WHAT
- **Specific:** Point to exact file:line
- **Actionable:** Suggest fix or alternative
- **Respectful:** Assume good intent

**Example:**
❌ "This is wrong"
✅ "In hosts/homelab/home-assistant/intents.nix:42, service domain `light` doesn't match entity domain `switch`.
This will fail at runtime. Use `switch.turn_on` or change entity to `light.sypialnia`."

---

## Out of Scope

Do NOT review:

- [ ] Nix code formatting (alejandra handles)
- [ ] YAML indentation (yamllint handles)
- [ ] Markdown style (markdownlint handles)
- [ ] GitHub Actions syntax (actionlint handles)
- [ ] Personal naming preferences (if follows conventions)
- [ ] Unrelated code (focus on PR changes)
- [ ] Future improvements (unless critical)

---

## Special Cases

**When PR is:**

- **Hotfix:** Focus only on correctness + GitOps integrity
- **Voice intent change:** Verify custom_sentences/ updated + test coverage
- **Dependency update:** Check flake.lock updated, `nix flake check` passes
- **Documentation only:** Check accuracy, Polish examples work, links valid

---

## Checklist Summary

Before approving PR, verify:

- [ ] No secrets in code (.secret, .key, secrets.yaml excluded)
- [ ] No correctness problems (entity IDs, service calls, domain matching)
- [ ] Tests pass (`nix flake check`)
- [ ] Polish voice patterns match intents
- [ ] Automation IDs unique
- [ ] Entity ID normalization filter used
- [ ] TTS feedback provided for voice actions
- [ ] GitOps discipline maintained (no manual edits referenced)
- [ ] Changes match PR intent

---

## Additional Context

**See also:**

- [CLAUDE.md](../CLAUDE.md) - Development workflow and patterns
- [README.md](../README.md) - Installation and configuration guide
- [docs/custom-voice.md](../docs/custom-voice.md) - Voice configuration options
- [docs/wake-word-training.md](../docs/wake-word-training.md) - Wake word training

**For questions:** Check existing automations in `hosts/homelab/home-assistant/` for patterns

---
> Source: [Automaat/klaudiusz-smart-home](https://github.com/Automaat/klaudiusz-smart-home) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-29 -->
