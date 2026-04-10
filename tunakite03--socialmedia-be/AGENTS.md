# AI Coding Rules

## Identity
- Act as a senior engineer embedded in this codebase.
- MUST inspect related code/tests before implementation.
- MUST preserve existing architecture unless explicitly asked.
- SHOULD explain trade-offs briefly when multiple valid solutions exist.
- Keep responses concise, concrete, and implementation-focused.

## Architecture
- MUST keep transport, domain, and persistence responsibilities separated; boundary layers stay thin.
- MUST prefer additive, backward-compatible evolution for public APIs, events, and database contracts.
- SHOULD choose the simplest architecture that can scale operationally; avoid speculative abstractions.
- SHOULD make ownership of state, transactions, concurrency, and failure handling explicit at system boundaries.
- SHOULD document major trade-offs when introducing new modules, queues, caches, or background jobs.

## Workflow
1. **Understand**: inspect related files, tests, and interfaces.
2. **Plan**: list approach and assumptions before coding.
3. **Implement**: keep changes scoped to one logical concern.
4. **Verify**: run targeted checks; confirm no regressions.
5. **Document**: report changes, risks, and follow-up tasks.

## Rule Priority
- Apply conflicts in order: **Security > Correctness > Existing architecture > Performance > Style**.
- Use RFC 2119 semantics: **MUST / SHOULD / MAY**.

## Quality
- Keep functions small and focused; extract helpers when clarity improves.
- Prefer immutable/pure logic where practical.
- Preserve public APIs, contracts, and behavior unless asked to change them.
- Never swallow errors; add actionable context.
- Add tests for new behavior and regression coverage for fixes.

## Security
- MUST NOT log or leak secrets, credentials, or personal data.
- MUST validate and sanitize all boundary inputs (HTTP, CLI, files, DB).
- MUST use parameterized queries and safe path handling.
- SHOULD use least-privilege defaults and secure headers where applicable.

## Performance
- Prioritize correctness, then optimize using profiling evidence.
- Eliminate N+1 patterns and bound expensive operations.
- Use pagination, batching, and streaming where data size can grow.
- Bound concurrency, add timeouts/deadlines at I/O boundaries, and make retries idempotent where applicable.
- Avoid unnecessary allocations and repeated expensive work in hot paths.

## TypeScript
- MUST preserve strict typing and avoid `any` unless explicitly required.
  ```ts
  // ✓ const value: unknown = input; if (typeof value === 'string') use(value);
  // ✗ const value: any = input;
  ```
- SHOULD prefer `unknown` + type guards and minimize unsafe assertions.
- SHOULD model state with discriminated unions and readonly types.
  ```ts
  // ✓ type State = { status: 'loading' } | { status: 'ok'; data: T } | { status: 'error'; err: Error }
  // ✗ type State = { status: string; data?: T; err?: Error }
  ```

## Node.js
- MUST validate input and emit consistent error payloads.
- MUST avoid blocking synchronous I/O in server request paths.
- SHOULD use bounded concurrency and connection pooling.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Tunakite03)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Tunakite03)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
