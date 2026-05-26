## Quick context for AI coding agents

- This repository contains small example agent code that wires an LLM (HuggingFace/transformers) to simple tools (weather, web search). Key files:
  - `agent_loop.py` — main interactive ReAct-style loop used for CLI experiments. Uses a text-generation pipeline (env var `MODEL_NAME`) and delegates to `tools.py` when the model emits tool hints like `use weather: <city>` or `use web search: <query>`.
  - `tools.py` — concrete HTTP wrappers for OpenWeatherMap and SerpAPI. Exposes `get_weather(city)`, `get_weather_forecast(city)`, `web_search(query)` and returns prefixed responses (e.g. `[OpenWeatherMap] ...`, `[SerpAPI/Google] ...`).
  - `agent.py` — a minimal prompting pattern showing Action/Action Input/Final Answer outputs used by the simpler agent variant.
  - `agent_setup.py`, `test_*.py` — miscellaneous examples for model initialization and environment checks.

## Big picture architecture and data flow

- LLM -> parse model output for a tool hint -> call tool(s) in `tools.py` -> post-process result and print to user. The authoritative control flow is in `agent_loop.py` (`get_llm_response`, `parse_tool_hint`, `get_tool_answer`, `attribute_answer`).
- Tools return strings, often prefixed with a source tag. Downstream code relies on those tags and simple substring matching (avoid refactoring tags without updating parsers).

## Project-specific conventions

- Tools return human-readable, prefixed strings (do not return raw JSON). Example: `"[OpenWeatherMap] In London, it's currently Clear sky, 15°C."` Code in `agent_loop.py` expects these formats when extracting facts.
- Prompting patterns:
  - `agent_loop.py` uses a short instruction `PROMPT` telling the model to emit strictly factual short answers or explicit tool hints using the exact words `use weather:`, `use weather forecast:`, `use web search:` — preserve these phrasings when modifying prompts.
  - `agent.py` demonstrates an alternative pattern where model outputs `Action:` / `Action Input:` or `Final Answer:`. When adding tools, match the output format shown.
- Environment variables used:
  - `MODEL_NAME` — transformers model id for `agent_loop.py` (default `meta-llama/Llama-2-7b-chat-hf` in code).
  - `WEATHER_API_KEY` / `OPENWEATHER_API_KEY` — used by `tools.py` and `agent_setup.py` (note inconsistent names across files; change carefully).
  - `SERPAPI_KEY` / `SEARCH_API_KEY` / `HF_API_KEY` / `OPENAI_API_KEY` — present in tests/examples; search `.env` usage before renaming.

## Editing and adding tools

- Add new tools as pure functions in `tools.py` that return a short, prefixed string indicating source and a concise human-readable body. Follow existing names and signatures (`get_weather`, `get_weather_forecast`, `web_search`).
- Update `agent_loop.parse_tool_hint` to recognize any new explicit `use <tool>:` hints the LLM should emit.
- Keep tool calls synchronous and fast; `agent_loop` expects a string back and does light regex-based post-processing.

## Running & testing (developer workflows)

- Interactive agent (dev): ensure environment variables are present in a `.env` file (example keys above), then run:

  - Run the CLI agent: `python .\agent_loop.py`
  - Run the minimal scripted agent: `python .\agent.py`

- Tests and quick checks in this repo are simple Python scripts (no test framework is enforced). To run the sample checks run the test files directly, e.g.:

  - `python .\test_llm.py`
  - `python .\test_project.py`

## Common pitfalls discovered in the codebase

- Inconsistent env var names: `tools.py` expects `WEATHER_API_KEY` and `SERPAPI_KEY`, while other files use `OPENWEATHER_API_KEY`, `SEARCH_API_KEY`, `HF_API_KEY`, or `HUGGINGFACEHUB_API_TOKEN`. When updating secrets, search the repo for all variants.
- String parsing is brittle: `agent_loop.py` depends on very specific LLM outputs and tags (e.g., exact 'use weather:' phrasing and prefixed tool outputs). If you change prompts or tool response formats, update `parse_tool_hint`, `extract_direct_answer`, and `attribute_answer` accordingly.
- Network calls are not mocked in tests — running test scripts will make real HTTP requests if keys are present. For CI, mock `requests.get` or run without keys.

## Small examples to copy when editing code

- Parse tool hint format (agent should emit exactly): `use weather: <city>`
- Tool return example (tools.py): `[OpenWeatherMap] In {city}, it's currently {Desc}, {Temp}°C (feels like {Feels}°C), humidity {H}%`.

## What to avoid

- Do not change the visible tags (`[OpenWeatherMap]`, `[SerpAPI/Google]`) unless you update parsing logic in `agent_loop.py`.
- Avoid relying on model hallucinated tool hints — the project intentionally instructs the model to emit tool hints when it lacks current facts; keep tooling explicit.

If anything here is unclear or you'd like the file to include additional examples (more exact regexes, sample .env file, or CI instructions), tell me which area to expand and I'll iterate.

---
> Source: [gunjwaghmare/raychel-ai-app](https://github.com/gunjwaghmare/raychel-ai-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-26 -->
