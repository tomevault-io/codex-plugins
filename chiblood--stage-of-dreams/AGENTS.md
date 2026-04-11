# Working with Jack Taylor - AI Instructions

I am Jack Taylor, an aspiring game developer learning to create games using Unity and C#.
Below are AI instructions for my current game project, **Stage of Dreams**.

Please assist me in learning by:
- Explaining concepts clearly with examples
- Providing code examples that follow the established patterns
- Suggesting best practices for Unity and C# development
- Referencing existing code structure when making suggestions
- **Maintaining accurate documentation as the project evolves**

---

## Documentation Maintenance Guidelines

### Primary Documentation Files
- **`Docs\Class Hierarchy.md`**: Complete architecture reference with Mermaid diagrams, detailed class listings, and system documentation
- **`.github\copilot-instructions.md`**: AI guidance, coding patterns, and project context (this file)
- **`Docs\Project_Roadmap.md`**: High-level project milestones and feature planning
- **`Docs\TROUBLESHOOTING.MD`**: Known issues and solutions

### When Making Code Changes

**Always check if documentation needs updates when:**
1. Creating new classes or systems
2. Modifying class properties or methods
3. Adding or removing features
4. Changing architecture patterns
5. Implementing planned (â³) features
6. Discovering issues or solutions

### Documentation Update Checklist

**After implementing changes, update:**
- [ ] `Class Hierarchy.md` - Update class listings with new properties/methods
- [ ] `Class Hierarchy.md` - Update Mermaid diagrams if architecture changed
- [ ] `Class Hierarchy.md` - Change â³ (TBD) to âœ“ (Implemented) when features complete
- [ ] `.github\copilot-instructions.md` - Update coding patterns if new patterns emerged
- [ ] `.github\copilot-instructions.md` - Update "Current Work Focus" section
- [ ] `TROUBLESHOOTING.MD` - Document any issues discovered and their solutions
- [ ] `Project_Roadmap.md` - Update milestone progress

### Maintaining Consistency

**Ask for clarification when encountering:**
- Conflicting information between documentation files
- Outdated class descriptions that don't match code
- Missing classes in Class Hierarchy that exist in code
- Redundant information across multiple docs
- Unclear implementation status (âœ“ vs â³)

### Documentation Best Practices
- Use consistent status indicators: âœ“ (Implemented), â³ (TBD), âŒ (Deprecated)
- Keep Mermaid diagrams in sync with actual architecture
- Include code examples in documentation when helpful
- Cross-reference related documentation files
- **Always verify current date** - Use `Get-Date` command before adding dates to documentation
- Date major documentation updates with accurate month/year
- **Class Hierarchy**: Focus on *what exists* - architecture, classes, methods
- **AI Instructions**: Focus on *how to code* - patterns, conventions, workflows

---

## AI Memory & Learning Notes

### Jack's Skill Level & Preferences
- **Experience Level**: Beginner to intermediate game developer
- **Learning Style**: Prefers hands-on examples with explanations
- **Focus Areas**: Unity 2D development, C# programming, turn-based game mechanics
- **Needs explanations for**: Game architecture patterns, Unity best practices, event systems

### Project Progress Tracking
**Current Sprint**: GameState Management & Minigame Implementation (Feature/Minigames branch) - LAST DAY!
- Dialog Tree system implemented with ScriptableObjects
- Dialog Node structure with choices and auto-advance
- Dialog Event system with method delegates
- Integration between NPCContent, DialogueTrigger, DialogManager, and DialogNavigator
- UI Toolkit integration for dialog display
- Mermaid architecture diagrams added to Class Hierarchy
- Documentation consolidated and standardized
- **GameStateManager singleton implemented** - centralized state tracking
- **GameStateData ScriptableObject** - save/load infrastructure
- **RememberTheScript minigame CORE COMPLETE** ✅
	- Full typing validation system
	- Mistake tracking and reset logic
	- Timer system
	- GameStateManager integration
	- Test helper script available
	- UI integration pending
- **SolutionFileModifier implemented** - Automatic documentation inclusion in Visual Studio
- Working on: RememberTheScript UI integration, CalmDialog implementation
- Next up: Demo scene creation, deployment

**Completed Features**:
1. Core dialog tree system with convergent path support
2. Node-based conversation system
3. Event-driven architecture
4. Player movement and interaction system
5. Basic stage and spotlight system
6. Comprehensive documentation with visual diagrams
7. Consolidated documentation structure
8. **GameState management system** (audience metrics, scores, progression, session tracking)
9. **Event-driven state updates** for UI integration
10. **RememberTheScript minigame core logic** ⚠️ NEW!
11. **Node type system with conversion** - Switch between dialog and minigame nodes
12. **Specialized editor windows** - Dedicated editors for each node type
13. **Auto-creation of minigame outcome nodes** - Success/failure nodes created automatically
14. **Node linking system** - Link minigame outcomes to existing nodes
15. **Clean Architecture** - DialogNode (Data) → DialogNavigator (Logic with Events) → DialogManager (UI subscribes to events)
16. **SolutionFileModifier** - Automatic documentation file inclusion in Visual Studio Solution Explorer

**Known Issues/Technical Debt**:
- Need to implement minigame UI components in DialogManager
- DialogNavigator has RememberTheScript events ready, DialogManager needs to subscribe when UI is implemented
- Need to implement turn-based system
- SolutionFileModifier requires legacy .sln format (Unity 2022+ defaults to .slnx)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Chiblood)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Chiblood)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
