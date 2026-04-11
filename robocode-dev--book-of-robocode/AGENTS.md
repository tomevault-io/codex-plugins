# Copilot Instructions for The Book of Robocode

## Core Guidelines

Follow the rules in `/AGENTS.md` for all content generation.

## Creating New Pages

Use the **create-page** skill to generate new book pages:

- **Slash command:** `/create-page <Page Name>`
- **Natural language:** "Create page: <Page Name>"

The page name must match an entry in `BOOK_STRUCTURE.md` (e.g., "Circular Targeting", "Wave Surfing Introduction").

The skill will:
1. Parse the hierarchy to derive slug, section, difficulty, and output path.
2. Gather sources from `specs/robowiki-links.md` (classic Robocode) and https://robocode.dev (Tank Royale).
3. Generate the page following `specs/page-generation-spec.md`.
4. Update `book/.vitepress/config.js` with sidebar and nav entries.

## Creating Illustrations

Use the **create-illustration** skill to generate SVG images from TODO markers:

- **Slash command:** `/create-illustration`
- **Natural language:** "Create illustration", "Generate the illustration", "Draw the SVG"

The skill scans the current file for `<!-- TODO: Illustration` markers and generates SVG files to `book/images/`.

The skill will:
1. Parse the TODO marker to extract filename, viewport, bots, lines, texts, etc.
2. Validate required fields (Filename, Caption, Viewport).
3. Generate the SVG with proper tank rendering and battlefield background.
4. Save the SVG to `book/images/<filename>`.
5. Insert an `<img>` tag below the TODO marker (keeping the TODO for regeneration).

## Reference Documents

- `/AGENTS.md` — Writing rules, terminology, page generation contract.
- `/BOOK_STRATEGY.md` — Audience, tone, and content philosophy.
- `/BOOK_STRUCTURE.md` — Complete table of contents with hierarchy and difficulty markers.
- `/specs/page-generation-spec.md` — Detailed frontmatter and body structure rules.
- `/specs/robowiki-links.md` — RoboWiki links for classic Robocode sources (not Tank Royale).
- `/.github/skills/create-page.md` — The create-page skill definition.
- `/.github/skills/create-illustration.md` — The create-illustration skill definition.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/robocode-dev)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/robocode-dev)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
