
# Project UI and AI conventions

When working on this codebase, **follow the conventions and element usage** defined in:

**`docs/AI-RULES-AND-UI-ELEMENTS.md`**

Consult that document whenever you:

- Add or change game boards, instructions, or voice/headline behavior
- Add or change buttons (use ReadyButton for "verify solution", Button for other actions)
- Add or change number inputs (use NumberInput, not raw `<input type="number">`)
- Add or change cards, pills, gradients, or feedback (FailureOverlay, FeedbackSuccessAnimation)
- Add new games or activities (Learn vs Game badge, HeadlineInstruction, CardLight)

The doc defines: instruction voice, HeadlineInstruction, ReadyButton, NumberInput, gradients, display of numbers (null/0 → 0 via toDisplayNumber/toDisplayValue), and general rules for AI-assisted changes. Keep changes consistent with it.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/polupravda)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/polupravda)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
