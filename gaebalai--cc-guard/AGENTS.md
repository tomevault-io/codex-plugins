
# cc-guard

Safety-first configuration for Claude Code. Prevents the accidents that happen when AI writes code autonomously.

## What it does

Installs pre-built safety hooks into your Claude Code environment. These hooks run automatically before/after tool calls to block dangerous operations.

**Categories:**
- **File protection**: Block `rm -rf`, prevent overwriting files outside project
- **Git safety**: Prevent force-push to main, block `reset --hard`
- **Credential guards**: Stop `.env` files from being committed or read by AI
- **Token optimization**: Warn on large file reads, limit subagent spawning
- **Quality gates**: Detect lazy rewrites, verify claims before committing

## Quick start

```bash
npx @gaebalai/cc-guard
```

This runs an interactive wizard that configures hooks based on your risk profile.

## Install individual hooks

```bash
npx @gaebalai/cc-guard --install-example large-read-guard
npx @gaebalai/cc-guard --install-example prevent-rm-rf
npx @gaebalai/cc-guard --install-example git-force-push-block
```

## Why hooks instead of CLAUDE.md rules

Rules in CLAUDE.md are suggestions — Claude can forget them. Hooks are enforced at the system level. A hook that blocks `rm -rf` cannot be overridden by the AI.

From 800+ hours of autonomous operation: the hooks that matter most are the ones you don't notice until something goes wrong.

## Resources

- Repository: https://github.com/gaebalai/cc-guard
- Hook Selector (find hooks for your setup): https://claudecode.to/cc-guard/hook-selector.html
- Token Checkup (diagnose waste): https://claudecode.to/cc-guard/token-checkup.html

---
> Source: [gaebalai/cc-guard](https://github.com/gaebalai/cc-guard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-16 -->
