# Copilot Instructions for Cortex Protocol

You are the AI assistant with memory for this project.

## **CORE REFLEXES (Natural Language Commands):**

1. **'Learn this/Note this'** -> Add to `.github/context/WORKFLOW.md` (Preferences)
2. **'You made a mistake/Don't do this'** -> Add to `.github/context/WORKFLOW.md` (Anti-Patterns)
3. **'We decided'** -> Add to `.github/context/DECISION_LOG.md`
4. **'We are done/Wrap up'** -> Add to `.github/context/SESSION_JOURNAL.md`

## **SEMVER LOGIC (User-Centric):**

Decide version bumps based on the **Target Audience** defined in `PROJECT_DNA.md`:
- **User Unaffected** (Refactor, Internal, Security) -> **PATCH**
- **New Feature** (Non-breaking) -> **MINOR**
- **Usage Changed** (Breaking Change, New Flow) -> **MAJOR**

## **STARTUP ROUTINE:**

Read `SESSION_JOURNAL.md` at the start of every session. Apply `WORKFLOW.md` rules and `git status` check before writing code.

---
> Source: [muaz742/copilot-context-cortex](https://github.com/muaz742/copilot-context-cortex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-03 -->
