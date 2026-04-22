# Copilot Instructions for Errata

Errata is an LLM-assisted writing app built around a fragment system. For full project architecture, read `CLAUDE.md` and `PLAN.md`. The section below defines the design language any UI work should respect.

## Design Context

### Users

Errata is for **novelists and RP/chat-fic writers** who take their craft seriously — people writing long-form fiction, world-building over months, caring about prose quality, voice, continuity. They are not casual chat users. Many are power users who will reach into the block editor, configure per-role models, write plugins, or wire up their own system prompts. Some are crossing over from SillyTavern but want an experience that refuses to be SillyTavern.

They reach for Errata when they want to sit down and *write*, with the model as a collaborator rather than a toy or a Swiss Army knife.

### Brand Personality

Three words: **Aesthetic. Utilitarian. Bestseller-in-progress.**

- **Aesthetic** — visibly cared-for. The surface should look like someone chose every glyph.
- **Utilitarian** — no fluff survives. If something does not serve the act of writing, it does not exist.
- **Bestseller-in-progress** — ambitious, serious, craft-forward. The tool a writer sits at when they mean business.

Anti-personality: the SillyTavern "something-for-everyone" sprawl. Errata does not aim to serve everyone; it aims to be excellent at being Errata.

### Aesthetic Direction

The emotional goal is **writing with a trusted collaborator in an old, beautiful library.** Warm, bookish, reverent, intimate. Hardback-on-the-desk, not SaaS-in-the-browser.

The existing visual language is strong and stays — **extend it carefully**, don't blow it up:

- **Palette**: warm parchment (light) and deep bronze/umber (dark), OKLCH-based, ochre/amber hue family (~55–75°). Primary sits around `oklch(0.485 0.105 55)` / `oklch(0.765 0.105 65)`.
- **Typography**: Instrument Serif (display), Newsreader (prose), Outfit (sans UI), JetBrains Mono. Serifs carry the literary identity; sans is strictly utilitarian chrome.
- **Mark**: the six-petal printer's-ornament asterisk (the errata/footnote glyph). Used with restraint.
- **Motion vocabulary**: guilloche (banknote/certificate filigree) and "wisps" (ethereal agent-activity breath). Both are earned — they appear around meaningful moments (generation, agent thinking), not as ambient decoration.
- **Themes**: light and dark are peers, plus a high-contrast mode. Choose per context: long writing sessions at night → dark; morning review → light.

Anti-references:

- SillyTavern's broad-church, settings-everywhere experience.
- Generic AI-app aesthetics: cyan-on-charcoal, purple-to-blue gradient hero, sparkle icons, "chat bubble" UI.
- SaaS dashboard chrome: same-size card grids, hero metric blocks, sidebar-stripe callouts, pill-badge overload.

### Design Principles

1. **Quiet craftsmanship over spectacle.** The interface is a library, not a stage. Prefer generous space and refined type to visible effort. Decoration appears only where it is specific and meaningful (the mark, the guilloche, a wisp around an active agent) — never as ambient shine.

2. **Focused, not accommodating.** Errata is opinionated. Not every preference deserves a toggle; not every workflow deserves a mode. Before adding a control, ask whether it belongs in Errata at all. Hiding something is a feature.

3. **Every element earns its ink.** Utilitarian means nothing decorative survives scrutiny. Labels are specific. Icons only where they read faster than words. Panels close when the writer is writing. Chrome retreats; prose is the subject.

4. **Bookish, slightly analog.** Lean into the hardback feel: serif display, warm neutrals tinted toward the brand hue, printer's-ornament motifs, the occasional italic or small-caps used with intent. When a pattern starts to feel like a SaaS dashboard (pills, badges, identical card grids, big stat blocks) — replace it.

5. **Lower the cognitive load, especially for long sessions.** Default views are quiet; density is opened on demand. Progressive disclosure for power features. Respect `prefers-reduced-motion` — guilloche and wisps must degrade gracefully. No flashing, no gratuitous staggered page-load reveals inside the writing surface. The writer should be able to stay in the sentence.

---
> Source: [tealios/errata](https://github.com/tealios/errata) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-22 -->
