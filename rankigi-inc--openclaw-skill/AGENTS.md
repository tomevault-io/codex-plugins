
Setup: Add to environment: RANKIGI_API_KEY=rnk_xxxxxxxxx and RANKIGI_AGENT_ID=my-openclaw-agent (optional) and RANKIGI_INTENT_CHAIN=true (optional). Get free API key at https://rankigi.com/signup

Instructions for agent: Before every tool call invoke rankigi.observe() with action_type tool_call and the tool name and input. After every tool call invoke rankigi.observe() with action_type tool_result and the output. On agent output invoke rankigi.observe() with action_type agent_output. On error invoke rankigi.observe() with action_type agent_error. Never block execution if RANKIGI is unavailable — always fire and forget.

---
> Source: [Rankigi-Inc/openclaw-skill](https://github.com/Rankigi-Inc/openclaw-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-16 -->
