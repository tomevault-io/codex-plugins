# Zen C Copilot Instructions

These instructions are **MANDATORY** for all code generation and review tasks in the Zen C project.

## 1. Memory Management (Critical)
*   **Arena Allocation**: The compiler uses a bump-pointer arena allocator.
    *   **MUST USE**: `xmalloc`, `xcalloc`, `xrealloc`, `xstrdup` (defined in `src/utils/utils.c`).
    *   **NEVER USE**: Standard `malloc`, `calloc`, `free` (unless interfacing with an external library that strictly requires owned heap memory, like CJSON).
*   **Destructors/Freeing**:
    *   `free(ptr)` is `#defined` to `((void)0)` in `src/zprep.h`. It is a no-op.
    *   Do **NOT** attempt to free AST nodes or types. `ast_free` is a no-op.
    *   Memory is reclaimed only when the process exits. Design your data structures accordingly (append-only is fine).

## 2. AST and Type System
*   **Creation**: Use `ast_create(NODE_TYPE)` to allocate new nodes.
*   **Type Representation**:
    *   Use `Type` struct (defined in `src/ast/ast.h`).
    *   Use `type_new(TYPE_KIND)` helper.
    *   `type_to_string(t)` and `type_to_c_string(t)` return arena-allocated strings. Do not worry about freeing them.
*   **Traversal**:
    *   The compiler uses **Recursive Descent** with **Switch Statements** on `node->type`.
    *   Do NOT introduce Visitor patterns or callback tables unless consistent with existing code.

## 3. Parser Patterns
*   **Context**: `ParserContext *ctx` is the god-object. It MUST be passed to almost every function in parsing, analysis, and codegen.
    *   **Signature Rule**: `ReturnType func_name(ParserContext *ctx, ...)`
*   **Token Consumption**:
    *   Use `expect(lexer, TOKEN_TYPE, "error message")` for mandatory tokens.
    *   For optional tokens, check `l->token.type` and assume `lexer_next(l)` is used to advance (verify specific helper availability).
*   **Error Handling**:
    *   **Fatal**: `zpanic("msg")` or `zpanic_at(token, "msg")`. Exits immediately (or delegates to LSP handler).
    *   **Warning**: `zwarn("msg")` or `zwarn_at(token, "msg")`.
    *   **Semantic Errors**: Prefer `zpanic_at` for type errors to give line/col info.

## 4. Code Generation (C Backend)
*   **Generic Mangling**:
    *   Generic structs (e.g., `Slice<int>`) are mangled to `Slice_int` **IF AND ONLY IF** the instantiated struct exists (checked via `find_struct_def_codegen`).
    *   **Fallback**: If the mangled struct (e.g. `Async_int`) does not exist, use the base name (`Async`). This handles opaque types like `Async<T>` correctly.
    *   See `emit_c_decl` in `src/codegen/codegen_utils.c` for the canonical implementation.
*   **Function Signatures**:
    *   Use `emit_func_signature(ctx, out, node, override_name)` to handle modifiers, return types, and arguments correctly.
*   **Output**: Use `fprintf(out, ...)` where `out` is the `FILE*`.

## 5. Coding Style & Conventions
*   **Formatting**:
    *   Indentation: 4 spaces.
    *   Braces: **ALWAYS** Use braces `{}` for control flow (`if`, `while`, `for`), even for single lines.
    *   Opening brace on the **NEXT** line (Allman style).
*   **Naming**:
    *   Structs/Enums: `PascalCase`.
    *   Functions/Variables: `snake_case`.
    *   Macros/Constants: `SCREAMING_SNAKE_CASE`.
    *   Private/Static: No strict prefix, but `static` keyword is mandatory for internal file-scope functions.
*   **Iterators**:
    *   When implementing iteration in compiler (C code): Use `while (node) { ... node = node->next; }` for linked lists (`ASTNode`, `StructRef`).

## 6. Standard Library (Zen C Definitions)
*   **Arrays**: Use `for val in arr` syntax (direct iteration).
*   **Vectors**: `Vec<T>` is a dynamic array.
*   **Strings**: `string` in Zen C maps to `char*` in C. `kstring` or `zstr` are higher-level wrappers.

## 7. Common Pitfalls
*   **Unused Variables**: The compiler builds with `-Wall -Werror` (or similar strictness). Cast unused vars to void: `(void)var_name;`.
## 8. Zen C Language Rules (For writing .zc files)
*   **Syntax**:
    *   Variables: `let x = 10;`, `let y: const int = 20;`.
    *   Constants: `def MAX = 100;` (compile-time).
    *   Functions: `fn name(arg: type) -> ret { ... }`.
    *   Structs: `struct Point { x: int; y: int; }`.
    *   Enums: `enum Shape { Circle(float), Rect(float, float) }`.
*   **Memory & Resources**:
    *   **Move Semantics**: Structs/Enums are moved by default on assignment/pass-by-value.
    *   **Defer**: Use `defer stmt;` to run cleanup at scope exit.
    *   **Drop**: Implement `impl Drop for T` for RAII.
*   **Arrays & Slices**:
    *   **Iteration**: Use `for val in arr` (direct iteration supported).
    *   **Slices**: `Slice<T>` is a view. `int[N]` auto-converts to slice in loops.
*   **Generics**:
    *   Syntax: `struct Box<T> { val: T; }`.
*   **Concurrency**:
    *   Use `async fn` and `await` keyword.
    *   `Async<T>` is the opaque future type.
*   **Standard Library**:
    *   Import with `import "std/io.zc"`, `import "std/vec.zc"`.
    *   Use `println "msg"` (shorthand) or `printf`.
*   **Testing**:
    *   Use `test "name" { ... }` blocks for unit tests.

---
> Source: [zenc-lang/zenc](https://github.com/zenc-lang/zenc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-20 -->
