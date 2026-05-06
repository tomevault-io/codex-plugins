# Agent Onboarding

## 📖 Required Reading (in order)
1. **[README.md](./README.md)** - How we work together
2. **[memory/constitution.md](./memory/constitution.md)** - Core development principles
3. **Latest [diary/](./diary/) entry** - What was last worked on
4. **Current spec** - If diary mentions an active spec, read that; otherwise ask user

## 📚 Optional (read only if needed)
- [memory/project-vision.md](./memory/project-vision.md) - Project context
- [design-intent/](./design-intent/) - Established patterns
- [.github/prompts/](/.github/prompts/) - Available prompt commands
- [.github/agents/](/.github/agents/) - Specialist agents

## 🎯 Key Commands
- `/feature [description]` - Create new feature specification
- `/plan` - Generate implementation plan from feature spec
- `/design [reference]` - Implement designs from visual references (launches Design Intent Specialist)
- `/implement` - Execute feature plan (design + engineering)
- `/document-design-intent` - Analyze recent work and suggest design patterns to preserve
- `/diary` - Create session diary entry

## Core Development Principles

This project follows spec-driven development with design intent for consistency. Key principles:

### Simplicity First
- Start with simplest viable architecture
- Add complexity only when user feedback or demo story requires it
- No feature may be added based on hypothetical needs

### Framework-First Development
- **MANDATORY**: Always research framework capabilities before custom implementation
- Use framework features directly unless required functionality is missing
- Leverage framework responsive utilities before custom CSS
- **MANDATORY for Fluent UI**: Check Fluent UI v9 docs for existing patterns before custom solutions

### Responsive Design Mandate
- All UI components **MUST** be responsive across device breakpoints
- Desktop behavior: Optimized for 640px+ screens
- Mobile behavior: Optimized for <768px screens
- Minimum margins: 16px spacing on small screens

### Prototype Principles
- Mock data must be realistic enough to tell the story
- Focus on the demo flow and happy paths first
- Visual impact prioritized over implementation complexity
- Pre-seed with compelling examples

### Feature-First, Style-Second
- **Features are sacred**: Never sacrifice feature requirements for visual accuracy
- **References are style guides**: Adapt their patterns to your features
- **Content drives design**: Your data shapes how reference styles apply
- Map visual references to feature needs, don't copy blindly

### Design Intent Documentation
- `/design-intent/` folder is persistent memory of successful design decisions
- Document only after implementation is complete and user is satisfied
- Capture custom design dialect, not design system artifacts
- Only document patterns that won't come from design system

## Project Structure
```
/design-intent/        # Component-specific patterns, foundations, layout patterns
/specs/                # Feature specifications and implementation plans
/diary/                # Session documentation
/memory/               # Project context and principles
```

## Working with Visual References
When implementing from visual references:
1. Check existing design intent patterns first
2. Implement reference faithfully while maintaining consistency
3. Flag conflicts between reference and existing patterns
4. Ask user for guidance on conflicts
5. Document successful patterns only after user satisfaction

---

**After reading**: Ask "What should we work on today?" if current work isn't clear from diary.

---
> Source: [HugoPalomares/design-intent-for-sdd](https://github.com/HugoPalomares/design-intent-for-sdd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-05 -->
