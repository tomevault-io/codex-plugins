
## Usage

```
coderlm <agent> --prompt <file> [--max-depth N] [--allowedTools TOOLS]
```

## Examples

```bash
echo "Find all TODO comments in src/" > task.txt
coderlm codex --prompt task.txt
coderlm "bunx --bun @google/gemini-cli" --prompt task.txt

echo "Fix type errors in src/" > task.txt
coderlm claude --prompt task.txt --allowedTools "Bash,Edit"
```

---
> Source: [CyrusNuevoDia/coderlm](https://github.com/CyrusNuevoDia/coderlm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-16 -->
