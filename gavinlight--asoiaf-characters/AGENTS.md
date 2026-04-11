# asoiaf-characters-2 Development Guidelines

Auto-generated from all feature plans. Last updated: 2025-11-05

## Active Technologies
- External Ice and Fire API integration with client-side caching (002-enhanced-character-search)
- External Ice and Fire API (https://anapioficeandfire.com) with server-side caching (003-book-references)
- Book appearances display with POV indicators (003-book-references)
- TypeScript (strict mode) with Node.js 18+ (001-character-info)
- External Ice and Fire API with server-side caching (30-minute TTL) (004-static-searchable-filters)

## Project Structure

```text
src/
tests/
```

## Commands

npm test && npm run lint

## Code Style

TypeScript (strict mode) with Node.js 18+: Follow standard conventions

## Recent Changes
- 004-static-searchable-filters: Added TypeScript (strict mode) with Node.js 18+
- 003-book-references: Implemented book appearances display, replaced TV series info with book context
  - Added BookAppearance interface and book-related fields to Character type
  - Implemented book transformation utilities (extractBookId, getBookTitle, mergeBookAppearances)
  - Added book appearances UI with POV indicators in CharacterBio.vue
  - Removed TV series and actor display sections
  - Non-sequential book IDs: 1,2,3,5,8 (A Game of Thrones through A Dance with Dragons)
- 002-enhanced-character-search: Added TypeScript (strict mode) with Node.js 18+

<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gavinlight)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/gavinlight)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
