# GitHub Copilot & AI Agent Instructions
TODO zzz Check if any of the content in this file is now outdated due to recent changes, and update if necessary. This file should be kept up-to-date with the current architecture and implementation strategy to ensure that AI agents have accurate guidance for contributing to the project.

## 1. Project Context & Copilot Instructions
You are assisting with the development of a high-performance **Macro Compiler and Text Expansion Engine**, which is intended to later be built as a custom node suite for **ComfyUI**. 

The engine uses a custom Orthogonal Syntax Matrix to parse, evaluate, and inject variables into natural-language generative AI prompts. It operates via an Abstract Syntax Tree (AST), a Double-Ended Context Stack, and a path-dependent deterministic PRNG.

**Current Implementation Status:** An initial test version of the core engine was implemented in `macro_engine.py` as a Python module with unit tests in `test_suite.py`, but it was dismantled and split into the actual implementation structure, consisting of a `core_engine.py` file containing the the outer function `string` -> `string` wrapper function and common definitions (eg `Token` and `ASTNode`), with the implementations of each separate subsystem of the engine (Lexer, Parser, Evaluator) split into separate files. `macro_engine.py` is left as a stub that redirects to the new architecture, but that is not yet completely reflected in the specifications. It should be removed once all references are converted to the new architecture. Files that relate to understanding or implementing specific decisions or changes to the project during development but that aren't part of the describing the current or intended state of the codebase or documentation, like plans for implementing a change or scripts used to update the codebase, are stored in the `.devTransient` folder to keep the top-level repo clean and focused on moving forward.

The architecture and implementation of the engine is documented across multiple specification files, with `UNIFIED_PROCESS_PLAN.md` describing the overall architecture strategy and rationale for specific design decisions and implementation requirements, `CORE_TYPES_AND_INTERFACES.md` describing shared data types and subsystem contracts, `LEXER_SPECIFICATION.md` describing the string → token lexing specifications, `PARSER_SPECIFICATION.md` describing the token → AST parser implementation roadmap, and `EVALUATOR_SPECIFICATION.md` describing the AST → string evaluation specifications. The standard base syntax for the input string is described `syntax.json`, and the other specification files should defer to `syntax.json` if there is confusion or conflict about the intended syntax. However, subsystems should be built generically to allow for changes or additions to the syntax characters to allow for customization and extensibility, but the core rules about the semantics and interactions of the various syntax can be treated as stable.

**Unified Parsing Architecture (Future):** The engine initially treated global context definitions and prompt text as separate inputs, but they should be unified into a single parsing pipeline. Both are subject to the same definition syntax and escaping rules. The roadmap is to merge `_parse_global_context` and `_lex_string` into a single character-by-character lexer that produces a mixed AST containing definition nodes and literal/invocation nodes, allowing definitions to appear anywhere in the input stream and creating true local scoping where definitions pushed during evaluation of one subtree do not leak to siblings unless the parent node is transparent. For the current Token-to-AST parser implementation plan and milestone sequence, refer to `PARSER_SPECIFICATION.md`.

**Specifications and Documentation:** The macro engine is documented across specialized specification files:
- [CORE_TYPES_AND_INTERFACES.md](../CORE_TYPES_AND_INTERFACES.md) - Shared data types and subsystem contracts (Token, ASTNode, Definition, MacroContext)
- [LEXER_SPECIFICATION.md](../LEXER_SPECIFICATION.md) - String → Token lexing specs (escape sequences, boundaries, definition lines)
- [PARSER_SPECIFICATION.md](../PARSER_SPECIFICATION.md) - Token → AST parser implementation roadmap
- [EVALUATOR_SPECIFICATION.md](../EVALUATOR_SPECIFICATION.md) - AST → String evaluation specs (7-phase lifecycle, context management)
- [UNIFIED_PROCESS_PLAN.md](../.devTransient/UNIFIED_PROCESS_PLAN.md) - Architecture strategy and migration plan from initial dual-pipeline implementation to unified pipeline

**Core Directives for AI Agents:**
* Prioritize deterministic execution. Generative AI prompts require exact repeatability based on seed and tree path.
* Do not attempt to use `re.sub` or raw Regex to parse Bounded Tokens (`< >`, `{ }`). You must use the custom Pushdown Automaton / character-by-character Lexer to prevent nesting failures.
* Maintain the strict separation between **State** (the Context Stack) and **Output** (the Return Buffer/Trace State Object). Do not allow literal text to leak into the definition contexts.
* When writing test cases, use raw strings (e.g., `r"string"`) for any strings containing regex patterns or escape sequences to avoid unintended Python string interpretation.
* Ensure syntax elements (e.g., `:`, `< >`, `/ /`) are not surrounded by extraneous whitespace in test strings or context definitions, as the parser is sensitive to exact formatting.
* For regex patterns in definitions, wrap both key and value in `/ /` if they contain regex syntax; use escape characters only for syntax markers, not arbitrary backslashes.
* When adding new tests, enable debug mode in `generate()` calls until the test passes to inspect stack and trace logs.
* Only unescape characters that are part of the defined syntax (e.g., `\` before `:`, `<`, `>`, `/`); do not arbitrarily skip backslashed characters outside of syntax.

## 1.1 Agent Operating Guidelines
* Use raw Python strings (`r"..."`) for regex and escape-heavy test input strings.
* Do not surround syntax markers with extraneous whitespace in tests or definitions.
* Enable debug mode on new tests until they pass, then remove or disable debug output.
* Do not delete existing comments unless they are directly replaced by equivalent documentation covering the same point.
* Do not create new markdown files for changes or summaries unless explicitly requested.

## 1.2 Comment and Concern Preservation
When refactoring, migrating, or removing code that contains user comments, TODOs, or documented concerns:

* **Acknowledge Explicit Concerns**: When removing comments containing complete thoughts, design decisions, TODOs, BUGs, or FIXMEs, explicitly acknowledge the removal in your chat response and explain how/where the concern was addressed (e.g., migrated to spec documentation, tracked as future work, resolved by implementation).
* **Document Migrations**: If a comment's core thought is rewording/relocating (e.g., moving to specification docs), a brief note in the response that the concern was migrated is sufficient.
* **Ambiguity Resolution**: When uncertain whether a comment is substantial, mention the removal in chat rather than silently deleting it. It's always safe to ask for clarification before removing.
* **Preserve Rationale**: Comments often capture decision rationale, edge case considerations, or alternative approaches. Future developers/agents rely on this history to understand why design choices were made.
* **Recovery**: While version history allows recovery, removals must be *known* to be discovered. Silent removal = lost knowledge.

**Example Response**: "Removed TODO about safe-mode switching (Milestone 4 comment) — this is tracked in PARSER_SPECIFICATION.md Milestone 6 under Error Handling as a future enhancement, with MAX_EVAL_DEPTH mentioned in EVALUATOR_SPECIFICATION.md."

---

## 2. Conventions and Behaviors to Preserve
When generating or refactoring code, adhere strictly to these architectural constraints:

### Scope & Stack Management
* **The Double-Ended Context Stack:** State definitions are managed in a `deque`.
  * Strong Definitions (`:`) push to the **Head** (Left).
  * Weak Definitions / Defaults (`::`) push to the **Tail** (Right).
* **Left-to-Right Traversal:** When searching for a variable definition, iterate forward through the context stack (normal order), ensuring Strong/local definitions are checked before Weak/global defaults.
* **Lexical Scoping:** Always push local scope markers/definitions when entering an AST node, and strictly `pop()` them when exiting, *unless* the node is invoked transparently (e.g., the `<|<Macro>>` special case).

### Execution Phasing
An AST Node lifecycle must strictly follow this order:
1. Push local arguments to Context. Perform rest of invocation key evaluation, lookup, and replacement TODO expand on that
  TODO Can this just be treated as pre-joining the invocation node token with the local args, and the numbered reference as strong definitions to the values of any definition arguments (defaulting `''`)? The syntax of the arguments should align perfectly with the syntax of context definitions, and I think the logic around the idea of transparent nodes aligns with this definition as well.
2. Apply Unbounded Pre-Patterns (execute Left-to-Right on the string, pulling patterns left-to-right from the stack in evaluation order).
3. Lex the string into Tokens of Literal text, Definitions, Bounded Invocations (eg `< >`), Bounded Groups (eg `{ }`), ValueSplits (eg `|`) and Modifier (eg `$$`).
4. Resolve Multi-Value sequences (`{a|b|c}`, `$$` logic).
5. Parse selected Tokens into Definitions and ASTNodes (Text or Invocation).
6. Recursively evaluate children and concatenate literal text.
7. Apply Unbounded Post-Patterns (Left-to-Right on the string, left-to-right evaluation from the stack).
8. Pop local scope.

### The Trace State Object
* Never implement multi-pass evaluation for secondary variables.
* The Root AST evaluates exactly once. Every bounded node resolution must be logged to a global `Trace State Dict`. Secondary ComfyUI outputs merely perform dictionary lookups against this trace.

---

## 3. Integration Notes (ComfyUI Specifics)
* **Node Mappings:** Ensure all classes are properly registered in `NODE_CLASS_MAPPINGS` and `NODE_DISPLAY_NAME_MAPPINGS`.
* **String Inputs:** Assume ComfyUI `STRING` widgets with `multiline=True`. Handle `\n` and `\r\n` safely during lexing.
* **Escape Characters:** The escape character (default `\`) should be pulled from node properties. Ensure it protects syntax markers during lexing and is stripped from the final Text Encode output.
* **LoRA Stripping:** Ensure `[group: <lora:...>]` wrappers are stripped before final output, as ComfyUI's native backend will throw errors on custom wrappers.

---

## 4. Top-Level Tasks for New PRs
When generating code for a new Pull Request, verify the following; for the latest Token-to-AST parser milestones and implementation details, refer to `PARSER_SPECIFICATION.md`.
1. **Implement or migrate parser parts according to `PARSER_SPECIFICATION.md`:**
   - `_parse_global_context` (context string syntax `:<`, `:`, `:>`).
   - `_lex_string` to return tokens+ASTNodes for `<...>` and `{...}` patterns.
   - `_push_local_args` to convert `<key|arg:val>` into context pushes.
2. **Keep behavior consistent with existing tests in `test_suite.py`** (`strong_shadowing`, `weak_defaults`, `lexical_scoping_isolation`, `transparent_dummy_root`, `unbounded_execution_order`).
3. **Lexer Robustness:** Does the Lexer handle escaped characters, malformed brackets (e.g., `< { > }`), and nested macros without catastrophic backtracking?
4. **Infinite Recursion Protection:** Is the global `MAX_EVAL_DEPTH` (or max node visit count) implemented to prevent stack overflows?

## 5. Workflow Commands
- Run tests from repo root: `python test_suite.py`
- Keep test assertions precise (no partial/magic string expectations unless explicit semantic intent).

## 6. What to Explain in Code Comments
- Explain each source of scoping effects (strong vs weak vs transparent) in `MacroContext` and `ASTNode._push_local_args` maps, and keep `PARSER_SPECIFICATION.md` aligned with the current parser design.
- Document the expected context/unified parser grammar in the current parser implementation and keep `PARSER_SPECIFICATION.md` in sync.

## 7. When to Ask for Human Input
- If behavior for nested conflicting regex substitution matches is unclear, ask for desired precedence (current test implies strong def wins in left-to-right stack order).
- If converting this to production, clarify the placeholder in `_lex_string` must become a stack-based balanced-delimiter parser, as tracked in `PARSER_SPECIFICATION.md`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GordonWalsh)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/GordonWalsh)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
