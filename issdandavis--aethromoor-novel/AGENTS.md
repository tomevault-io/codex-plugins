# The Avalon Codex - AI Coding Agent Instructions

## Project Overview
This is "The Avalon Codex" - a multi-generational fantasy narrative project centered around **Polly's Wingscroll: The First Thread**, a choice-based interactive game set in the Spiral of Pollyoneth universe.

## Essential Architecture

### Dual Game Structure
- **HTML Version** (`game/index.html`) - Complete, playable browser game with 30+ scenes and 14 endings
- **ChoiceScript Version** (`choicescript_game/`) - Professional mobile game framework for app store publication
- Both versions tell the same story with different technical implementations

### Content Organization
- **`lore/`** - Fantasy worldbuilding (characters, magic systems, geography)
- **`writing_drafts/`** - Novel manuscripts and narrative content
- **`game/`** - Complete HTML game (40,000+ words)
- **`choicescript_game/`** - Professional game framework (partial implementation)
- **`docs/`** - Project roadmaps and automation guides

## Critical Patterns

### The Spiral of Pollyoneth Universe
- **Central Character**: Izack Thorne (mage-turned-king who founded Avalon Academy)
- **Key Companion**: Polly ("Polydimensional Manifestation of Accumulated Wisdom and Occasional Sarcasm")
- **Core Magic System**: Collaborative casting vs. hierarchical magical control
- **Narrative Structure**: Four-generation epic spanning 150+ years

### Game Development Workflow
1. **HTML First**: Content prototyped in browser-based version (`game/`)
2. **ChoiceScript Conversion**: Professional scenes built in `choicescript_game/scenes/`
3. **Cross-Reference Required**: HTML and ChoiceScript versions must maintain story parity

### File Naming Conventions
- **Chapter Files**: Use descriptive titles (`# Chapter 1 The Cave and the Contra.txt`)
- **Character Chronicles**: Named by protagonist (`IZACK_MASTER_CHRONICLE_UPDATED.txt`)
- **Game Scenes**: Descriptive lowercase (`first_lesson.txt`, `singing_dunes.txt`)

## Development Guidelines

### When Working on Game Content
- Always check both HTML (`game/`) and ChoiceScript (`choicescript_game/`) versions
- Maintain consistency between character names, stats, and story beats
- Reference `docs/PROJECT_ROADMAP.md` for current development phase
- Test narrative choices lead to appropriate stat changes (Collaboration, relationships)

### When Working on Lore
- Cross-reference character details in `IZACK_MASTER_CHRONICLE_UPDATED.txt`
- Maintain consistency with established magic system (dimensional theory, collaborative casting)
- Check character relationships span multiple generations correctly

### Repository Navigation
- **Start Here**: `START_HERE.md` for quickest orientation
- **Play Game**: `game/index.html` (double-click to play)
- **File Locations**: `FILE_LOCATIONS.txt` for quick reference
- **Development Status**: `docs/PROJECT_ROADMAP.md`

## Integration Points

### External Dependencies
- **ChoiceScript IDE** for professional game development
- **Zapier Workflows** (documented in `docs/AUTOMATION_GUIDE.md`)
- **GitHub Pages** for HTML game deployment
- **Hosted Games Platform** for mobile publication

### Key Relationships
- HTML game scenes → ChoiceScript scene conversion
- Character lore → Game dialogue consistency
- Statistical progression → Narrative outcome branching
- Multi-generational timeline → Character relationship continuity

## Quality Standards
- **Word Count**: 40,000+ words across all game content
- **Narrative Paths**: 14 unique endings with meaningful choice consequences
- **Character Consistency**: Cross-reference character details across all documents
- **Technical Parity**: HTML and ChoiceScript versions tell identical stories

## Current Priority
Phase 2 development: Converting remaining HTML expeditions (`singing_dunes.txt`, `verdant_tithe.txt`, `rune_glacier.txt`) to ChoiceScript format while maintaining narrative consistency and implementing proper stat tracking.

## Multi-AI Collaboration Guide
Use multiple assistants in clearly defined roles to avoid overlap:
- **Lore Curator (Claude/creative model)**: Validates new narrative against `lore/` and `IZACK_MASTER_CHRONICLE_UPDATED.txt`; flags timeline or magic rule conflicts.
- **Conversion Engineer (Copilot / Continue)**: Translates HTML scene content in `game/scenes/` into ChoiceScript `.txt` files under `choicescript_game/scenes/` preserving choice logic and stats.
- **Structural Reviewer (Cody / Codeium)**: Ensures scene parity (same branching & ending counts) and checks stat progression consistency with `startup.txt` and `choicescript_stats.txt`.
- **Automation Planner (Any)**: Updates Zapier workflow docs in `docs/AUTOMATION_GUIDE.md` when new content types added (e.g., expedition assets).
- **Quality Balancer**: Runs passes to equalize Collaboration stat difficulty; compares thresholds across expeditions.

### Shared Context Artifacts
Maintain these lightweight anchors so every AI has the same baseline:
- `STATUS_CONTEXT.md` (create if missing): Weekly snapshot: current scene being converted + pending lore updates.
- `SCENE_PARITY_CHECKLIST.md`: List each HTML scene vs ChoiceScript counterpart with status (Missing / Draft / Verified).
- `STATS_MATRIX.md`: Table of each choice affecting Collaboration or relationships (source file + delta).

### Collaboration Workflow
1. Curator approves a scene's lore integrity (HTML source).
2. Conversion Engineer drafts ChoiceScript scene (no stat tuning yet).
3. Structural Reviewer verifies branching & endings alignment; updates checklist.
4. Quality Balancer adjusts stat thresholds; updates `STATS_MATRIX.md`.
5. Automation Planner notes any new triggers for analytics or Zapier.

### Hand-off Conventions
- Add `// TODO:[ROLE]: description` inline only in ChoiceScript draft files; remove before verification.
- Prefix commit messages with role tag: `Lore:`, `Convert:`, `Struct:`, `Balance:`, `Auto:`.

### Model Selection Tips
- Creative/lore tasks: higher reasoning (Claude / GPT creative variant).
- Structural/stat tasks: deterministic/code-focused (Copilot / Codeium).
- Cross-file parity checks: tools with codebase indexing (Cody / Continue).

### Quick Checks Before Merging
- Scene has same number of choices and endings references as HTML original.
- No new character traits contradict existing chronicles.
- Stat changes logged in `STATS_MATRIX.md`.
- Roadmap phase still accurate; update `docs/PROJECT_ROADMAP.md` if scope shifted.

---
> Source: [issdandavis/aethromoor-novel](https://github.com/issdandavis/aethromoor-novel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-24 -->
