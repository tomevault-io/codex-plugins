Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

## 5. Style And Architecture Defaults

**Respect project conventions and prefer testable design.**

- Follow `.editorconfig` rules for formatting, spacing, naming, and file-level style.
- Do not introduce style changes that conflict with `.editorconfig` unless the user explicitly asks.
- Avoid static classes and static state as much as possible.
- Prefer dependency injection with interfaces and constructor injection for services and cross-cutting behavior.
- Use static members only when they are truly stateless utilities or constants.

## 6. Maintainability Principles: DRY, SOLID, KISS

**Use these as default decision filters when writing or modifying code.**

- **DRY (Don't Repeat Yourself):** avoid duplicating business rules, validation logic, and transformation logic.
- If similar logic appears in more than one place, extract a shared function/class at the same abstraction level.
- Duplicate literals and magic values should become named constants when reused.
- Do not over-abstract one-off code just to "be DRY"; prefer local clarity first.

- **SOLID:** design code so behavior can evolve without fragile coupling.
- **S - Single Responsibility:** each class/module should have one clear reason to change.
- **O - Open/Closed:** add behavior through composition or new implementations, not by repeatedly editing stable code.
- **L - Liskov Substitution:** derived types must be safely usable through base contracts without special-case checks.
- **I - Interface Segregation:** prefer small focused interfaces over broad "god" interfaces.
- **D - Dependency Inversion:** depend on abstractions; wire concrete implementations at composition boundaries.

- **KISS (Keep It Simple, Stupid):** choose the simplest solution that satisfies current requirements.
- Prefer straightforward control flow over clever patterns.
- Avoid speculative extensibility and premature optimization.
- If a solution is hard to explain in a few sentences, simplify it.

- **Conflict resolution rule:** when principles compete, apply them in this order: KISS, then DRY, then SOLID.
- **Review check:** before finalizing, verify new code is understandable, minimally duplicated, and easy to extend safely.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

---
> Source: [nicolaparo/BlazorThree](https://github.com/nicolaparo/BlazorThree) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-17 -->
