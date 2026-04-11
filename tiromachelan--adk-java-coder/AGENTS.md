# ADK Java Coder — Copilot Instructions

This repository is a **Python ADK (Agent Development Kit) project** that autonomously develops Java programs from a natural-language task. It uses OpenAI models via the OpenAI API (routed through LiteLLM).

## Build & Run

```bash
# Install dependencies
pip install -r requirements.txt

# Set up environment (copy and fill in OPENAI_API_KEY)
cp .env.example .env

# Run the agent pipeline
python main.py "Write a Calculator class with add, subtract, multiply, divide"
```

No test suite for the Python code itself — the agents test the *generated* Java code using JUnit5.

## Architecture

```
SequentialAgent (java_coder)             ← pipeline.py
 LlmAgent: first_version              ← agents/first_version_agent.py
 LoopAgent: tdd_loop (max 20 cycles)
    └── SequentialAgent: tdd_cycle
        ├── LlmAgent: test_writer        ← agents/test_writer_agent.py
        ├── LlmAgent: test_runner        ← agents/test_runner_agent.py
        └── LlmAgent: code_improver      ← agents/code_improver_agent.py
```

- **Loop exit:** `test_runner` outputs `ALL_TESTS_PASSED` when all JUnit5 tests pass; the LoopAgent also stops at 20 iterations.
- **LLM backend:** `gpt-5-mini` via `OPENAI_API_KEY`. ADK routes OpenAI calls through LiteLLM (`openai/gpt-5-mini` model string).
- **Java toolchain:** plain `javac` / `java` — no Maven or Gradle. JUnit5 standalone jar is downloaded to `workspace/lib/` on first run.

## Key Conventions

### Adding a new agent
1. Create `agents/my_agent.py`, define an `LlmAgent` instance with `name`, `model`, `instruction`, and `tools`.
2. Import and wire it into `pipeline.py`.

### Tools
All tools are `google.adk.tools.FunctionTool` wrappers around plain Python functions in `tools/shell_tools.py`. They operate on the `workspace/` directory. Add new tools there and import them into the relevant agent file.

### Workspace
`workspace/` is gitignored and recreated at runtime. All generated `.java` files, compiled `.class` files, and the JUnit5 jar live here. The `WORKSPACE` path constant is defined in `tools/shell_tools.py`.

### Loop exit signal
`test_runner_agent` must emit the string `ALL_TESTS_PASSED` in its response text for the LoopAgent to escalate and exit early. If you change this signal, update both `agents/test_runner_agent.py` (the prompt) and `pipeline.py` (the `_all_tests_passed` check).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Tiromachelan)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Tiromachelan)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
