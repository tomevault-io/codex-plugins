# Pokemon Champions Calculator — Architecture Rules

## ABSOLUTE RULES — DO NOT VIOLATE

### 1. Data Access: champions.ts is the ONLY gateway to @smogon/calc data
- **NEVER** import `Generations` from `@smogon/calc` in any file except `src/data/champions.ts`
- **NEVER** create a `gen9` variable anywhere except `src/data/champions.ts`
- **NEVER** access `gen9.species`, `gen9.types`, `gen9.items`, `gen9.moves`, or `gen9.abilities` directly
- **ALWAYS** use the exported functions: `getAvailablePokemon()`, `getPokemonData()`, `getAvailableItems()`, `getAvailableMoves()`, `getAvailableAbilities()`, `getTypeEffectiveness()`, `getDefensiveMultiplier()`
- These functions filter out non-Champions data (legendaries, mythicals, paradox Pokemon, CAP fakemon, banned forms, non-Champions items)

### 2. Sprites: sprites.ts is the ONLY place that constructs sprite URLs
- **NEVER** construct a Showdown sprite URL inline (e.g., `` `https://play.pokemonshowdown.com/sprites/ani/${id}.gif` ``)
- **NEVER** write your own species-name-to-sprite-ID conversion regex
- **ALWAYS** use `getSpriteUrl(species)` for animated sprites
- **ALWAYS** use `getSpriteFallbackUrl(species)` for static fallback sprites
- The converter handles: Mega-X/Y/Z naming, missing Z-A Mega sprite fallbacks, regional forms

### 3. Champions-Specific Restrictions
- **No Terastallization** — Tera is NOT in Champions. The `teraType` field exists in PokemonState for @smogon/calc compatibility but must always be `''`
- **No Legendaries/Mythicals/Paradox/Ultra Beasts** — all excluded in `EXCLUDED_POKEMON` set in champions.ts
- **Limited item pool** — only items in `CHAMPIONS_ITEMS` whitelist are available (no Life Orb, Choice Band/Specs, Assault Vest, Heavy-Duty Boots, etc.)
- **SP system replaces EVs/IVs** — 66 total SP, 32 max per stat. All Pokemon have 31 IVs equivalent
- **Mega Evolution only battle gimmick** — no Z-Moves, no Dynamax
- **Some moves are restricted** — Return, Scald removed. Knock Off not on all Pokemon (confirmed missing from Incineroar)

### 4. State Management in PokemonPanel
- **NEVER** use `useCallback` with `state` in the dependency array — causes stale closure bugs
- When building a new PokemonState, **ALWAYS** start from `createDefaultPokemonState()` and set each field explicitly
- **NEVER** spread old state (`{ ...state, key: value }`) across multiple sequential calls — use a single `onChange()` with the complete new state
- `onApplySpread` must set SPs AND nature in a single `onChange()` call

### 5. Live Data Filtering
- All data from Smogon VGC 2026 stats (`pkmn.github.io/smogon/data/stats/gen9vgc2026.json`) must be filtered before display
- **Pokemon**: filter through `getAvailablePokemon()` — removes legendaries, paradox, etc.
- **Items**: filter through `getAvailableItems()` — removes Life Orb, Choice Band, etc.
- **Teammates**: filter through `getAvailablePokemon()` before showing partner suggestions
- The VGC 2026 meta is DIFFERENT from Champions — data is directionally useful but must be adapted

### 6. Archetype System
- "Optimize for Meta" button MUST use the same `getArchetypes()` function that the Archetypes panel displays
- One system, one answer — the user should see the same result whether they click Optimize or pick the first archetype
- Presets provide moves/ability, archetypes provide SP spreads + nature

## File Architecture

```
src/data/champions.ts    — SINGLE gateway to @smogon/calc. All Pokemon/item/move/type data
src/utils/sprites.ts     — SINGLE source for all sprite URLs
src/types.ts             — Shared TypeScript types (PokemonState, FieldState, etc.)
src/data/presets.ts      — Curated competitive sets (nature, SPs, moves, items)
src/data/tierlist.ts     — Static tier rankings (Game8 + community)
src/data/teams.ts        — Pre-built team compositions
src/data/faqs.ts         — SEO FAQ content
src/data/liveData.ts     — Fetches + caches Smogon usage stats
src/data/synergies.ts    — Algorithmic partner recommendation engine
src/calc/championsCalc.ts — Damage calc wrapper
src/calc/spOptimizer.ts   — Role-based SP spread suggestions
src/calc/archetypes.ts    — Multi-archetype detection + move matching
src/calc/metaBenchmarks.ts — Damage calcs vs meta threats
src/calc/teamAudit.ts     — Team gap analysis
src/calc/teamBuilder.ts   — Incremental team construction
src/calc/matchupAnalysis.ts — Head-to-head matchup scoring
src/calc/metaDiscovery.ts  — Original strategy discovery
src/calc/metaDeduction.ts  — Champions meta projection
src/calc/itemOptimizer.ts  — Team-aware item suggestions
```

## Confirmed Champions Data Sources
- **Game8** (game8.co/games/Pokemon-Champions) — tier lists, movesets, items, builds
- **Smogon stats** (pkmn.github.io/smogon/data/stats/gen9vgc2026.json) — usage data (filtered for Champions)
- **Showdown sprites** (play.pokemonshowdown.com/sprites/ani/) — animated sprites
- Pokemon in Champions: ~251 fully evolved + Pikachu, no legendaries/mythicals
- Items: type-boost (20%), berries, Focus Sash, Choice Scarf, Leftovers, Mega Stones

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/taylorsmithgg) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
