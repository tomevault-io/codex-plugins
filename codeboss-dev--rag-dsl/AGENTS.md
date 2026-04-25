# PromptScript for RAG — Implementation Plan

## Context

This project builds **PromptScript**, a lightweight typed DSL for constructing RAG prompts, and benchmarks it against equivalent plain-English prompts across 50 standardized RAG tasks. The goal is to empirically test whether structured prompt construction improves faithfulness, format compliance, and token efficiency compared to freeform English prompts.

---

## Project Structure

```
CoSamplingPlayground/
├── pyproject.toml
├── CLAUDE.md
├── .gitignore
│
├── src/promptscript/
│   ├── __init__.py
│   ├── grammar.lark              # Lark EBNF grammar
│   ├── parser.py                 # Lark-based parser
│   ├── ast_nodes.py              # Dataclass AST node definitions
│   ├── transformer.py            # Parse tree -> AST
│   ├── type_checker.py           # Static type validation
│   ├── compiler.py               # AST -> PromptSegments -> output
│   ├── token_budget.py           # tiktoken-based budget enforcement
│   ├── targets/
│   │   ├── __init__.py
│   │   ├── markdown.py           # Markdown prompt renderer
│   │   └── json_api.py           # JSON API body renderer (OpenAI/Anthropic)
│   ├── runtime/
│   │   ├── __init__.py
│   │   ├── context.py            # Variable bindings, retriever stubs
│   │   └── builtins.py           # Built-in functions (retriever.fetch, len)
│   └── cli.py                    # CLI: promptscript compile/check/tokens
│
├── benchmark/
│   ├── dataset/
│   │   ├── corpus/               # Source documents (JSON)
│   │   └── tasks.jsonl           # 50 tasks with ground truth
│   ├── prompts/
│   │   ├── plain_english/        # 50 plain-English prompts (.txt)
│   │   └── promptscript/         # 50 PromptScript files (.ps)
│   ├── retriever/
│   │   ├── __init__.py
│   │   └── bm25.py               # BM25 retriever (rank_bm25)
│   ├── runner.py                 # LLM API orchestration
│   ├── scorer.py                 # 4-metric scoring
│   ├── pipeline.py               # End-to-end evaluation
│   └── config.yaml               # Models, retriever params, paths
│
├── report/
│   ├── analysis.py               # Tables, charts, statistical tests
│   ├── figures/
│   └── report.md
│
└── tests/
    ├── test_parser.py
    ├── test_transformer.py
    ├── test_type_checker.py
    ├── test_compiler_markdown.py
    ├── test_compiler_json.py
    ├── test_token_budget.py
    ├── test_scorer.py
    └── fixtures/                 # .ps files + expected outputs
```

---

## Phase 1: DSL Grammar + Compiler

### 1a. Grammar (`grammar.lark` using Lark EBNF)

**Why Lark:** EBNF syntax, built-in Transformer pattern for tree-to-AST, LALR backend for speed, large community.

Key grammar rules:
- **Declarations**: `type_spec IDENT "=" expression` — types are `str`, `int`, `float`, `bool`, `persona`, `instruct`, `context[]`
- **Parameters**: `set_param IDENT "=" literal` — routes to API envelope, not prompt body
- **Control flow**: `for var in expr { ... }` and `if condition { ... } else { ... }` — braces over indentation for unambiguous parsing
- **Compile call**: `prompt.compile(args...)` — explicit compilation trigger
- **Triple-quoted strings**: `"""..."""` for multi-line instruct blocks
- **Dotted names**: `retriever.fetch(...)` without needing a full object system
- **Comments**: `// single line`

### 1b. AST Nodes (`ast_nodes.py`)

Dataclass-based: `Program`, `Declaration`, `Assignment`, `SetParam`, `ForLoop`, `IfBlock`, `CompileCall`, plus expression types (`StringLiteral`, `NumberLiteral`, `BoolLiteral`, `Identifier`, `FuncCall`, `ListExpr`, `Condition`).

### 1c. Transformer (`transformer.py`)

Lark `Transformer` subclass — maps parse tree nodes to AST dataclasses. ~120-150 lines.

### 1d. Type Checker (`type_checker.py`)

Single-pass AST visitor with a symbol table. Validates:
- Type consistency (e.g., `int x = "hello"` is an error)
- `context[]` must come from a function call returning a list
- `set_param` values must be numeric or boolean literals
- All `prompt.compile()` args must reference declared variables
- Loop variable typing (`for chunk in docs` where `docs: context[]` gives `chunk: context`)

### 1e. Compiler (`compiler.py`)

Two-phase design:
1. **Evaluation**: Walk AST, resolve identifiers, expand loops/conditionals, produce flat list of `PromptSegment(role, content, token_count, metadata)`
2. **Rendering**: Dispatch segments to the selected target renderer

### 1f. Token Budgeting (`token_budget.py`)

Uses `tiktoken` (cl100k_base). When total tokens exceed budget, drops `context[]` segments from lowest confidence first (whole chunks, not mid-chunk truncation). Logs dropped chunks.

### 1g. Render Targets

- **`targets/markdown.py`**: Structured markdown with `## Role`, `## Context`, `## Query`, `## Instructions` sections
- **`targets/json_api.py`**: Ollama/OpenAI-compatible messages array; `set_param` values map to top-level API params (works with Ollama's `/v1/chat/completions` endpoint)

### 1h. CLI (`cli.py` via Click)

```
promptscript compile input.ps --target markdown|json --output out --context context.json
promptscript check input.ps          # parse + type-check only
promptscript tokens input.ps         # token counts per segment
```

---

## Phase 2: Benchmark Dataset (50 tasks)

### Task Distribution

| Task Type | Count |
|---|---|
| Factual QA | 15 |
| Multi-hop Reasoning | 12 |
| Summarization with Citations | 13 |
| Out-of-Context Detection | 10 |

### Corpus

Wikipedia paragraphs from Natural Questions + SQuAD 2.0 passages (for unanswerable tasks). ~200 chunks stored as JSON in `corpus/`.

### Task Format (`tasks.jsonl`)

Each task: `task_id`, `task_type`, `query`, `relevant_doc_ids`, `ground_truth` (answer, citations, answerable flag), `difficulty`, `metadata`.

### Prompt Pairs

For each task, create both a plain-English `.txt` and a PromptScript `.ps` file containing **identical information content** — only structural formatting differs.

---

## Phase 3: Evaluation Pipeline

### Key Design: Fixed Retriever

Both prompt types receive the **exact same retrieved documents** per task. BM25 retriever runs once; output is shared. This isolates prompt format as the sole independent variable.

### Components

- **`retriever/bm25.py`**: BM25 via `rank_bm25`, deterministic, no GPU
- **`runner.py`**: Calls local Ollama model via its OpenAI-compatible API (`http://localhost:11434/v1`), configurable via `config.yaml`
- **`scorer.py`**: Four metrics, each [0, 1]:
  1. **Answer Correctness**: 0.4 * relaxed exact match + 0.6 * ROUGE-L F1
  2. **Faithfulness**: Claim-level ROUGE overlap against retrieved chunks (no second LLM needed)
  3. **Format Compliance**: Task-type-specific checks (length, citations, refusal phrases)
  4. **Token Efficiency**: `prompt_tokens / (correctness + 0.01)`, normalized
- **`pipeline.py`**: End-to-end orchestration: retrieve -> compile -> call LLM -> score -> write results

---

## Phase 4: Report + Analysis

- **Summary table**: Mean/std for each metric, per prompt type, per task type (2x4x4)
- **Statistical test**: Paired Wilcoxon signed-rank (non-parametric, N=50)
- **Visualizations**: Grouped bar charts, scatter plots (correctness vs faithfulness), token histograms, failure mode breakdown
- **Failure case analysis**: Manual categorization of divergent results
- All figures generated programmatically via `matplotlib` for reproducibility

---

## Key Technical Decisions

| Decision | Choice | Rationale |
|---|---|---|
| Parser | Lark (EBNF) | Transformer pattern, LALR, well-maintained |
| Block delimiters | Braces `{}` | Avoids indentation-parsing complexity |
| Compiler targets | Both markdown + JSON | Human review + API calls |
| LLM | Local Ollama model | Free, reproducible, no API costs |
| Token counting | tiktoken (cl100k_base) | Approximate counts, widely used baseline |
| Retriever | BM25 (rank_bm25) | Deterministic, reproducible, no GPU |
| Statistical test | Wilcoxon signed-rank | Non-parametric, paired, N=50 |
| Faithfulness metric | Claim-level ROUGE | No second LLM dependency |

---

## Dependencies

**Core**: `lark`, `tiktoken`, `click`, `pyyaml`
**Benchmark**: `openai` (for Ollama's OpenAI-compatible API), `rank-bm25`, `rouge-score`, `matplotlib`, `pandas`, `scipy`
**Dev**: `pytest`, `pytest-cov`, `ruff`

---

## Verification

1. **Unit tests**: ~40-50 tests across parser, transformer, type checker, compiler, token budget, scorer
2. **Grammar validation**: All 50 `.ps` files compile without errors
3. **Dataset validation**: JSON schema check on `tasks.jsonl`, all doc IDs exist in corpus, all tasks have both prompt files
4. **Integration test**: Full pipeline on 3 tasks with mock LLM responses
5. **End-to-end**: Run benchmark, inspect results JSONL, verify statistical analysis output

---

## Resolved Decisions

- **Compiler targets**: Both markdown and JSON built from the start
- **LLM**: Local Ollama model (no cloud API costs, fully reproducible)
- **JSON format**: Ollama's OpenAI-compatible format (`/v1/chat/completions`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CodeBoss-dev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
