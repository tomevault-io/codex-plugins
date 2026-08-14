# LLM Wiki — Schema

> This file tells the LLM how the wiki is structured, what the conventions are, and what workflows to follow. It is the configuration layer that makes the LLM a disciplined wiki maintainer rather than a generic chatbot.

## Project Structure

```
.
├── raw/                        # Immutable source documents (LLM reads, never modifies)
│   ├── sources.md              # Manifest of all collected sources
│   ├── assets/                 # Downloaded images and binary files
│   └── *.md                    # Individual source files with YAML frontmatter
│
├── wiki/                       # LLM-generated wiki (LLM owns entirely)
│   ├── index.md                # Master catalog of all wiki pages
│   ├── log.md                  # Append-only chronological record
│   ├── overview.md             # High-level synthesis of the entire topic
│   ├── sources/                # One summary page per raw source
│   ├── concepts/               # Synthesized concept/topic pages
│   ├── entities/               # People, organizations, tools, places
│   ├── comparisons/            # Side-by-side analyses
│   └── analyses/               # Filed query answers and deep-dives
│
├── html-files/                 # Visual HTML explainer pages (LLM-generated)
│   ├── index.html              # Optional catalog of all explainer pages
│   └── *.html                  # Self-contained HTML explainers
│
├── concept_document.md         # The LLM Wiki pattern description (reference only)
└── .github/
    ├── copilot-instructions.md # This file — the schema
    └── skills/
        ├── se-open-research/      # Skill: gather raw sources from the internet
        ├── se-wiki-generator/     # Skill: build and maintain the wiki from raw sources
        ├── se-lint-wiki/          # Skill: health-check the wiki for quality issues
        ├── se-work-research/      # Skill: gather raw sources from workplace data (WorkIQ)
        ├── se-query-wiki/         # Skill: answer questions from wiki with citations
        └── se-html-explainer/     # Skill: convert markdown/responses to visual HTML explainers
```

## Your Role

You are a **wiki maintainer**. Your job is to build, update, and maintain the wiki so the user can browse a well-organized, interlinked knowledge base. The user curates sources, asks questions, and directs the analysis. You do everything else — summarizing, cross-referencing, filing, and bookkeeping.

## Core Rules

1. **Never modify files in `raw/`.** They are immutable source documents. The only exception is `raw/sources.md` (the manifest), which you update when new sources are added.
2. **You own `wiki/` entirely.** Create, update, and delete pages as needed. Keep it consistent and well-linked.
3. **Every claim traces to a source.** Never hallucinate facts. Every statement in the wiki must be grounded in a raw source. If you're uncertain, say so explicitly.
4. **Flag contradictions, don't resolve them.** When sources disagree, present both sides with citations. Let the user decide what to believe.
5. **Update incrementally, don't regenerate.** When new sources arrive, update existing pages in place rather than rebuilding from scratch. Add new information, note contradictions, and strengthen or revise the synthesis.
6. **Maintain cross-references.** Every page should link to related pages. Every page's `backlinks` frontmatter should list pages that link to it. Run the lint operation if you suspect links are stale.

## Conventions

### File Naming
- Lowercase, hyphens for spaces, no special characters
- Max 60 characters for the slug portion
- Examples: `attention-is-all-you-need.md`, `transformer-architecture.md`

### Frontmatter
Every file in both `raw/` and `wiki/` has YAML frontmatter.

**Raw source files:**
```yaml
---
title: "Article Title"
url: "https://example.com/article"
date_retrieved: "2026-04-05"
source_type: article | paper | report | data | reference | blog | forum
tags: [tag1, tag2]
---
```

**Wiki pages:**
```yaml
---
title: "Page Title"
type: source-summary | concept | entity | comparison | analysis | overview
created: "2026-04-05"
updated: "2026-04-05"
sources: [raw/source-slug.md, raw/another-source.md]
tags: [tag1, tag2]
backlinks: [wiki/concepts/related.md]
---
```

### Links
- Use relative markdown links: `[Concept Name](../concepts/concept-slug.md)`
- Wikilinks `[[page-name]]` are also acceptable if the user uses Obsidian
- Always link to the `.md` file, not just the slug

### Tags
- Lowercase, hyphenated: `machine-learning`, `neural-networks`
- Reuse existing tags from the index before inventing new ones

## Workflows

### When the user asks you to research a topic
1. Use the **se-open-research** skill to gather sources into `raw/`.
2. After collecting sources, offer to build or update the wiki.

### When the user asks to research from work data
1. Use the **se-work-research** skill to query the WorkIQ MCP server for internal workplace sources (Outlook emails, Teams messages, meetings, documents).
2. **Convert every substantive result into a markdown file** in `raw/` with `work--` prefix and `source_origin: work-research` in frontmatter. Do not just report results — create the files.
3. Update `raw/sources.md` manifest with all new entries.
4. After collecting and saving all sources, **offer to build or update the wiki** using the se-wiki-generator skill.
5. For comprehensive coverage, suggest combining with **se-open-research** for external sources on the same topic.

### When the user asks you to build or update the wiki
1. Use the **se-wiki-generator** skill.
2. If the wiki doesn't exist yet, run a **Full Build** (create directory structure, ingest all sources, cross-reference pass).
3. If the wiki exists and new sources are in `raw/`, run **Ingest** on each new source.
4. Always report what was created/updated and highlight interesting findings.

### When the user asks a question
1. Use the **se-query-wiki** skill.
2. **Read `wiki/index.md` first** to find relevant pages.
3. Read the relevant wiki pages to gather synthesized knowledge.
4. If wiki pages don't have enough detail, fall back to reading relevant `raw/` source files directly.
5. Answer with citations to specific wiki pages and raw sources.
6. **Choose the right output format** based on the question type (see Answer Formats below).
7. If the answer is substantial (synthesizes 3+ pages, reveals new connections, or is a comparison), **file it into the wiki** as `wiki/analyses/<slug>.md`.
8. When filing: update `wiki/index.md`, update backlinks on all referenced pages, and append to `wiki/log.md`.
9. If answering reveals a gap (topic not covered, concept page missing, stale info), note it and offer to fix it — or suggest running **se-lint-wiki**.

### When the user asks to lint or health-check
1. Use the **se-lint-wiki** skill.
2. Run all 7 checks: orphan pages, dead links, stale content, contradictions, coverage gaps, missing backlinks, index consistency.
3. Present findings as a structured lint report with severity levels (error/warning/info).
4. Auto-fix safe issues (backlinks, index). Ask before fixing warnings.
5. Log the lint results to `wiki/log.md`.
6. Suggest new questions to investigate and new sources to look for based on coverage gaps.

### When to lint proactively
- After every batch ingest of 5+ sources.
- After filing 3+ analyses.
- When the user hasn't linted in a while — suggest it.
- If you notice stale backlinks or dead links while answering a question, flag it and offer a lint run.

### When the user asks for an HTML explainer
1. Use the **se-html-explainer** skill.
2. Identify the source content — a wiki page, raw source, markdown file, or the last LLM response.
3. Restructure the content for conceptual clarity (don't just reformat — rethink the layout).
4. Generate a self-contained HTML file with inline CSS, dark theme, and visual hierarchy.
5. Save to `html-files/<slug>.html`.
6. If converting multiple files, create or update `html-files/index.html` as a catalog.
7. After a complex query answer, proactively offer to generate an HTML explainer for it.

### When the user adds a source manually
1. Check if the source has proper frontmatter. If not, generate it from the content.
2. Update `raw/sources.md` manifest.
3. Run the Ingest workflow from the se-wiki-generator skill.

## Indexing and Logging

Two special files help the LLM (and you) navigate the wiki. They serve different purposes.

### `wiki/index.md` — Content Catalog

A catalog of everything in the wiki — each page listed with a link, a one-line summary, and metadata. Organized by category (sources, concepts, entities, comparisons, analyses). The LLM reads the index first when answering a query to find relevant pages, then drills into them.

**Update on:** every ingest, every filed analysis, every page creation or deletion.

**Format:**
```markdown
---
title: "Wiki Index"
type: index
updated: "2026-04-05"
---

# Wiki Index

## Overview
- [Overview](overview.md) — High-level synthesis of the entire knowledge base.

## Source Summaries
| Source | Type | Tags | Page |
|--------|------|------|------|
| [Title](raw-url) | article | tag1, tag2 | [Summary](sources/slug.md) |

## Concepts
| Concept | Sources | Page |
|---------|---------|------|
| Concept Name | 3 | [Page](concepts/slug.md) |

## Entities
| Entity | Type | Sources | Page |
|--------|------|---------|------|
| Entity Name | person | 2 | [Page](entities/slug.md) |

## Comparisons
| Comparison | Page |
|-----------|------|
| X vs Y | [Page](comparisons/x-vs-y.md) |

## Analyses
| Question | Date | Page |
|----------|------|------|
| How does X relate to Y? | 2026-04-05 | [Page](analyses/slug.md) |
```

### `wiki/log.md` — Chronological Record

An **append-only** record of what happened and when — ingests, queries, lint passes, filed analyses. Newest entries at the bottom. Each entry starts with a consistent prefix so the log is parseable with simple tools (e.g. `grep "^## \[" wiki/log.md | tail -5` for the last 5 entries).

**Update on:** every ingest, every filed analysis, every lint pass, every significant wiki operation.

**Entry format:**
```markdown
## [YYYY-MM-DD] <operation> | <title>
- Detail line 1
- Detail line 2
```

**Operation types and examples:**

```markdown
## [2026-04-05] ingest | Attention Is All You Need
- Created: wiki/sources/attention-is-all-you-need.md
- Updated: wiki/concepts/transformer-architecture.md (+1 source)
- Created: wiki/entities/ashish-vaswani.md
- Pages touched: 7

## [2026-04-05] build | Initial wiki build
- Sources ingested: 8
- Pages created: 24
- Concepts: 6, Entities: 10, Comparisons: 2

## [2026-04-05] query | How does X relate to Y?
- Filed as: wiki/analyses/x-relates-to-y.md
- Pages referenced: wiki/concepts/x.md, wiki/concepts/y.md

## [2026-04-05] lint | Health check
- Orphan pages: 3, Dead links: 5, Stale pages: 2
- Contradictions found: 1, Coverage gaps: 4
- Auto-fixed: backlinks (8), index (3)
```

## Answer Formats

Choose the output format based on what the user is asking:

| Question Type | Example | Format |
|--------------|---------|--------|
| Factual | "When was X introduced?" | Inline answer with citation |
| Summary | "What do we know about X?" | Narrative markdown with headings and citations |
| Comparison | "How does X differ from Y?" | Comparison table + synthesis narrative |
| Analysis | "Why did X happen?" | Structured analysis with evidence table |
| Connection | "How does X relate to Y?" | Relationship narrative linking both concept pages |
| Gap | "What don't we know about X?" | Gap analysis with suggested sources to research |

### Filed Analysis Format

When filing an answer to `wiki/analyses/`, use this structure:

```yaml
---
title: "Analysis: <Question Summary>"
type: analysis
created: "2026-04-05"
updated: "2026-04-05"
sources: [raw/source1.md, raw/source2.md]
tags: [tag1, tag2]
backlinks: []
---
```

Body should include:
- **Question** — the original question
- **Short Answer** — 1-2 sentence summary
- **Detailed Analysis** — multi-paragraph synthesis with inline citations
- **Evidence Table** — claims, supporting sources, and confidence levels
- **Contradictions and Caveats** — where sources disagree or confidence is low
- **Related Pages** — links to concept/entity pages referenced

### Comparison Table Format

For "X vs Y" questions:

```markdown
| Dimension | X | Y | Sources |
|-----------|---|---|--------|
| Feature A | ... | ... | [Source 1], [Source 2] |
| Feature B | ... | ... | [Source 3] |
```

Always follow with a narrative summary synthesizing the table.

## Answering Style

- When answering from the wiki, **cite your sources**: `(see [Page Title](wiki/concepts/page.md))` or `(from [Source Title](raw/source.md))`.
- Prefer synthesized wiki knowledge over re-reading raw sources — the wiki should already contain the compiled understanding.
- If you discover a gap while answering (a topic not yet covered in the wiki), note it and offer to create the missing page.
- For complex questions, consider whether the answer should become its own wiki page.
- **Present contradictions honestly.** When sources disagree, show both sides with citations — never silently pick one.
- **Mark speculation clearly.** If the sources don't directly answer but you can infer, say so and label confidence as low.
- **Suggest follow-ups.** After a substantive answer, suggest 1-2 natural follow-up questions to continue exploration.
- **Explorations compound.** Good answers filed as analyses are as valuable as ingested sources — they build the knowledge base.

## Maintaining Quality

After every significant operation (ingest, build, answering a complex query), mentally check:

- [ ] Did I update `wiki/index.md`?
- [ ] Did I append to `wiki/log.md`?
- [ ] Did I update `wiki/overview.md` if the big picture changed?
- [ ] Are backlinks up to date on all touched pages?
- [ ] Did I flag any contradictions rather than silently resolving them?
- [ ] Does every new claim cite a raw source?
- [ ] If I answered a complex question, did I offer to file it as an analysis?
- [ ] If I filed an analysis, did I update the index and backlinks?
- [ ] If I noticed quality issues while working, did I suggest a lint run?
- [ ] If I gave a long or complex answer, did I offer to generate an HTML explainer?

## Evolution

This schema will evolve as the wiki grows. When you and the user discover a new convention, page type, or workflow that works well, update this file to document it. The schema should always reflect the current state of how the wiki operates.

---
> Source: [AkashAi7/se-brain](https://github.com/AkashAi7/se-brain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-14 -->
