- Agent messages (`agent_messages` table) always use Neon main branch, never task-specific branches
- Messages are agent coordination, not task data - they persist across branch experiments
- Query messages from `main` branch (spring-field-87079189), not task branches
- only query messages on user request 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EmilyFlocke-Pierce)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/EmilyFlocke-Pierce)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
