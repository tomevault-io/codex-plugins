# GitHub Copilot Instructions

> Workspace context for GitHub Copilot Chat and inline suggestions

## Repository Type: Tool Collection ("Junk Drawer")

This is a **multi-tool repository** containing self-contained utilities. Each top-level folder is an independent tool with its own README, dependencies, and implementation.

## Critical: Auto-Generated README

**The main README.md is auto-generated.** To update it:

```bash
# Run the generator
node .github/scripts/generate-readme.js

# Script sources content from:
# - manualMappings object in generate-readme.js (curated descriptions)
# - Tool folder discovery (auto-detects folders with README.md)
```

## Repository Structure

```
.
├── README.md                    # Auto-generated (don't edit manually)
├── .claude.md                   # AI agent instructions
├── [tool-folder]/              # Each tool is self-contained
│   ├── README.md               # Tool docs (required)
│   └── [implementation]        # Tool code
└── .github/
    ├── workflows/
    │   └── update-readme.yml   # Auto-updates README on push
    └── scripts/
        └── generate-readme.js  # README generator
```

## Adding New Tools

### 1. Create Tool Folder

Format: `lowercase-with-hyphens`

### 2. Required: Tool README.md

Template:
```markdown
# Tool Name

**One-line compelling description**

## Purpose
What it does, why it exists, who needs it

## Prerequisites
- Required tools/versions
- Optional dependencies

## Installation
Step-by-step setup

## Usage
With code examples

## Stack
**Stack:** Languages, frameworks, tools
```

### 3. Add Curated Description

Edit `.github/scripts/generate-readme.js`, add to `manualMappings`:

```javascript
'tool-name': {
  name: 'tool-name',
  title: 'Display Name',
  subtitle: 'One-line hook',
  description: 'What it does and why.',
  features: ['Feature 1', 'Feature 2', 'Feature 3', 'Feature 4'],
  stack: 'Node.js, Python, etc.',
  emoji: '🚀'
}
```

### 4. Test Locally

```bash
node .github/scripts/generate-readme.js
git diff README.md  # Verify output
```

## Current Tools

| Tool | Description | Stack |
|------|-------------|-------|
| `agentic-assessment` | AI development impact analysis from git history | PowerShell, Python, Node.js, GitHub CLI |
| `time-narrative` | Clockify → AI-generated weekly narratives | Node.js, SQLite, Claude Code |

See curated descriptions in `generate-readme.js` for reference.

## Tool Philosophy

Include tools that:
- ✅ Solve real problems elegantly
- ✅ Are production-ready and documented
- ✅ Are self-contained
- ❌ Don't need independent versioning
- ❌ Don't need separate CI/CD

## Code Suggestions

### When suggesting code for tools:
- Match the tool's existing language/framework
- Use modern syntax (ES6+, Python 3.8+, PowerShell 7+)
- Include error handling
- Add comments for complex logic
- Follow tool's existing patterns

### For new tools:
- Prefer cross-platform when possible
- Use standard libraries over heavy dependencies
- Include comprehensive README with examples
- Add dependency files (package.json, requirements.txt, etc.)

### For generator scripts:
- Node.js with no external dependencies (keep it simple)
- Write idiomatic JavaScript
- Handle edge cases gracefully
- Maintain existing code style

## Common Patterns

### Emoji Selection (in generate-readme.js)
```javascript
const emojiMap = {
  'agentic|assessment|analyzer': '🤖📊🔬',
  'time|log|monitor': '⏱️📝📡',
  'api|webhook': '🔌🪝',
  'deploy|pipeline': '🚀🔁'
};
```

### Feature Lists
- 3-4 items
- Concrete capabilities
- Include impressive stats when available
- Technical but accessible language

### Stack Format
Comma-separated, key technologies only:
```
Node.js, Python (matplotlib), Azure CLI
```

## Don't Suggest

- Manual edits to main README.md (use generator)
- Experimental or unfinished tools
- Tools with cross-dependencies on other tools in repo
- Committing secrets or credentials
- Global dependencies without documentation

## Helpful Commands

```bash
# Test README generation
node .github/scripts/generate-readme.js

# List all tools
ls -Directory | Where-Object { Test-Path "$($_.Name)/README.md" }

# Check for uncommitted changes
git status

# Preview README changes
git diff README.md
```

## Priority Files

When asked about the repo, reference:
1. `.claude.md` - Full AI agent context
2. `CONTRIBUTING.md` - Contribution guidelines
3. `.github/README.md` - Auto-generation system docs
4. Individual tool READMEs - Tool-specific docs

## GitHub Actions

Workflow: `.github/workflows/update-readme.yml`
- Triggers on push to main/master
- Runs generate-readme.js
- Commits changes if README updated
- Prevents loops (ignores README.md and .github/** changes)

---

**Keep the drawer organized! 🗄️**

---
> Source: [JeremyVyska/copilot-junk-drawer](https://github.com/JeremyVyska/copilot-junk-drawer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-04 -->
