# CLAUDE.md

This file provides guidance to Claude Code and GitHub Copilot when working with code in this repository.

## Environment Setup

This repository contains Claude Code settings, configurations and guidance. The default setup uses GitHub Copilot as the model provider through a proxy API.

### Required Dependencies

- `copilot-api`: Install globally with `npm install -g copilot-api`
- Run `copilot-api start --proxy-env` to authorize GitHub Copilot account
- Use tmux for session management: `tmux new-session -d -s copilot 'copilot-api start --proxy-env'`

### Configuration

- `settings.json`: Contains environment variables for API configuration
- Uses `localhost:4141` as the API base URL
- Configured to use `claude-sonnet-4.5` as the primary model
- Telemetry and non-essential traffic are disabled

## Skills

Skills are reusable capabilities defined in the `skills/` directory. Each skill has a `SKILL.md` file with YAML frontmatter containing `name` and `description` fields. The description serves as both documentation and trigger condition.

### Skill File Structure

All skill files must include YAML frontmatter:

```yaml
---
name: skill-name
description: 'Comprehensive description that also serves as the trigger condition. Include keywords and phrases that should activate this skill.'
---
```

### Directory Structure

```sh
skills/
├── reflection/          # Session analysis and CLAUDE.md improvement
├── eureka/              # Technical breakthrough documentation
├── translate/           # Tech article translation to Chinese
├── command-creator/     # Create Claude Code custom commands
├── github-fix-issue/    # Fix GitHub issues end-to-end
├── github-review-pr/    # Review GitHub pull requests
├── skill-creator/       # Create and benchmark agent skills
├── codex-skill/         # Handoff tasks to Codex CLI
├── autonomous-skill/    # Long-running task automation
├── nanobanana-skill/    # Image generation with Gemini
├── kiro-skill/          # Interactive feature development
├── spec-kit-skill/      # Constitution-based development
├── deep-research/       # Multi-agent research orchestration
└── youtube-transcribe-skill/  # YouTube transcript extraction
```

### Usage

Skills are invoked via slash syntax or triggered automatically:

```sh
/skill-name [arguments]
```

**Examples:**

- `/reflection deep` - Comprehensive session analysis
- `/eureka "Discovered batching optimization"` - Document a breakthrough
- `/translate [text or file]` - Translate to Chinese

### Skill Development Principles

When creating or modifying skills:

- **Comprehensive descriptions**: The `description` field triggers the skill, so include relevant keywords and phrases
- **Structure content clearly**: Use clear sections with step-by-step instructions
- **No `$ARGUMENTS`**: Skills receive user input through natural conversation, not via variable substitution
- **Define specific outputs**: Include explicit output formats and structures
- **Keep skills focused**: One skill, one purpose

### Behavioral Guidelines

- **Concise communication**: Provide direct answers without unnecessary preamble or elaboration
- **Follow existing patterns**: Always check similar skills for consistent structure and approach
- **Prefer editing over creating**: Always edit existing files rather than creating new ones unless absolutely necessary
- **Use TodoWrite for complex tasks**: Track multistep processes and ensure completion of all requirements

## Guidances

Claude Code guidances should be put under `guidances/` directory and linked at README.md.

---
> Source: [feiskyer/claude-code-settings](https://github.com/feiskyer/claude-code-settings) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-04 -->
