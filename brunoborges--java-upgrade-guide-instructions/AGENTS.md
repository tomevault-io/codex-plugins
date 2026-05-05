# Copilot Instructions

## Repository Purpose

This repository provides GitHub Copilot instruction files for upgrading Java projects across JDK versions. The instruction files are **not code to be edited**—they are reference content derived from official JEP (Java Enhancement Proposal) documentation scraped from OpenJDK.org.

Users copy the appropriate `java-*-to-*.instructions.md` file into their own project's `.github/copilot-instructions.md` to get upgrade guidance from Copilot.

## Architecture

- **Instruction files** (`java-8-to-11.instructions.md`, `java-11-to-17.instructions.md`, `java-17-to-21.instructions.md`, `java-21-to-25.instructions.md`): Copilot instruction sets covering language features, API changes, and migration patterns for each upgrade path. These are the primary deliverables.
- **Upgrade checklists** (`upgrade-checklist-*.md`): Step-by-step migration checklists for each upgrade path.
- **Scrapers** (`scrapers/`): Node.js scripts that extract JEP documentation from OpenJDK.org. Each scraper (`scraper-11-17.js`, `scraper-17-21.js`, `scraper-21-25.js`) produces a corresponding `jep-documentation-*.json` file used to generate the instruction files.

## Scraper Commands

All scraper commands run from the `scrapers/` directory:

```bash
cd scrapers
npm install
npx playwright install   # required for browser-based scraping
npm run scrape            # runs the scraper (see package.json "scrape" script)
```

## Conventions

- Instruction file names follow the pattern `java-{source}-to-{target}.instructions.md` (e.g., `java-21-to-25.instructions.md`).
- Checklist file names follow `upgrade-checklist-{source}-{target}.md`.
- Scraped JSON data lives in `scrapers/jep-documentation-{source}-{target}.json`.
- Each instruction file is organized by JEP number with migration patterns showing old vs. new code side by side.
- All JEP content must originate from official OpenJDK sources—do not fabricate or hallucinate JEP details.

---
> Source: [brunoborges/java-upgrade-guide-instructions](https://github.com/brunoborges/java-upgrade-guide-instructions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-05 -->
