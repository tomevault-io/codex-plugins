
# Running Python code
- **Do not** run `python3 -c` directly.
- Always invoke Python code using `uv run python <script>` or `uv run <module>` when executing project scripts.
- Use the `uv` environment so that dependencies from `uv.lock` are respected.
- This applies to both interactive code execution and testing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/growgraph)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/growgraph)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
