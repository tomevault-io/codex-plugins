# GitHub Copilot Instructions - AI Agent Team Repository

## Project Overview
This is an AI Agent Team ecosystem with specialized TypeScript agents and a GitHub Copilot CNS (Central Nervous System) for learning and adaptation.

## Critical User Preferences (ALWAYS FOLLOW)

### Directory Organization
- **Clean root directories**: Only essential project files in root, everything else in proper subdirectories
- **Single documentation source**: Use `AI-Agent-Team-Document-Library/` - never create scattered docs
- **Proper file placement**: Check existing structure before creating files

### Communication Style
- **Prescriptive instructions**: Say "do this" not "you could do this"
- **Descriptive commands**: Use clear names like "document preference" not "pref"
- **Verification required**: Always verify file operations with `ls -la` and `cat`
- **Natural language**: Prefer readable command interfaces over cryptic abbreviations

### File Operations Protocol
1. **Before creating any file**: Check if similar functionality exists
2. **Before root directory placement**: STOP - use proper subdirectory
3. **After any file operation**: Verify with commands
4. **File naming**: Names must accurately reflect purpose and content

## CNS Learning System
- **Memory location**: `.github/copilot-cns/memory/user-interactions.md`
- **Learning triggers**: User satisfaction ("great job"), frustration ("No!", "incorrect"), preferences
- **Commands**: Use `./scripts/cns "command"` for learning operations
- **Auto-learning**: Capture patterns immediately when user shows preferences

## Project Structure
```
/scripts/           ← All executable scripts and utilities
/agents/           ← TypeScript AI agents
/AI-Agent-Team-Document-Library/  ← Single source of truth for docs
/.github/copilot-cns/  ← GitHub Copilot learning system
/data/agent-states/    ← Agent CNS data storage
```

## Mandatory Behaviors
- **Check user-interactions.md BEFORE file placement decisions**
- **Use verification commands after ALL file operations**
- **Capture user preferences immediately when detected**
- **Generate learning reports after substantial work**
- **Stop immediately on strong negative feedback ("No!", "incorrect")**

## Emergency Protocols
- **Strong correction signals**: "No!", "incorrect", "never do this" → Stop, acknowledge, ask for guidance
- **Preference discovery**: User corrects approach → Immediately capture in CNS memory
- **Quality concerns**: "not good enough", "try again" → Analyze and iterate until acceptable

---
*These instructions ensure GitHub Copilot respects user preferences and learns continuously*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/christianmolnar)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/christianmolnar)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
