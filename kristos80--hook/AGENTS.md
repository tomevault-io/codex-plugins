# CLAUDE.md

## Testing

Coverage and mutation testing must both pass at 100%. Always run both after making changes.

```bash
# Coverage
herd coverage ./vendor/bin/pest --coverage

# Mutation testing
herd coverage ./vendor/bin/pest --mutate --parallel --everything --covered-only
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kristos80)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/kristos80)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
