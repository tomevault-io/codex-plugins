# Copilot Review Instructions for 3ngram (engram)

These instructions apply to all GitHub Copilot code reviews in this
repository. Read them in full before reviewing any pull request.

## 1. Project Overview

3ngram is an agentic RAG memory system — a kernel-governed memory OS for
AI agents. The Python package name is **engram** (source in `src/engram/`,
Python 3.12+).

The system provides typed, versioned memory with eight record types:
Belief, Decision, Episode, Skill, Entity, Preference, Reflection, and
Resource. It features temporal awareness, HippoRAG-inspired retrieval,
and a hybrid Librarian Gate for durable memory writes.

Key architecture decisions:

- **Monolith-first** (ADR-001): single Python service with clear module
  boundaries. No microservices.
- **Dual protocol** (ADR-002): MCP for agent-to-tools communication, A2A
  v0.3 for agent-to-agent communication.
- **Postgres + pgvector** (ADR-003): unified relational and vector
  storage. No separate vector database.
- **Swappable embedding adapter** (ADR-007): interface abstraction with
  fastembed as the default backend.
- **Expanded agent architecture** (ADR-008): RL Gate Advisor, Research
  Scanner, Meta-Optimizer. Agent roles defined as Python Protocol classes.
- **Knowledge graph** (ADR-009): NetworkX for prototyping, Neo4j as a
  future migration target.
- **HippoRAG retrieval** (ADR-011): 6-stage pipeline with adaptive
  routing (pattern separation, indexing, retrieval, reranking, context
  assembly, response generation).
- **Hybrid Librarian Gate** (ADR-012): declarative rules combined with an
  RL-trained advisor to govern memory writes.
- **Self-healing pattern**: aiobreaker circuit breakers and tenacity
  retries at module boundaries.

## 2. Design-First Enforcement

This project follows a strict design-first development model (STD-020).
Implementation code is not authorized without approved technical specs
and schema definitions.

**The SDLC phases are sequential and mandatory:**

1. Initiation (complete)
2. Design (complete)
3. Specification (current phase)
4. Implementation (not yet authorized)
5. Verification
6. Release
7. Operations

**Review rules:**

- Flag any PR that introduces implementation code (under `src/engram/`)
  without referencing an approved technical specification or schema
  definition in the PR description.
- Design and specification artifacts are the only authorized deliverables
  right now. Code changes limited to project config, CI, tooling, and
  test scaffolding are acceptable.
- If a PR adds code that implements business logic (memory operations,
  retrieval pipelines, agent behavior, storage adapters), it MUST cite
  the specific technical spec (by document ID) that authorizes it.
- Skeleton modules with only `__init__.py` exports, type stubs, or
  Protocol definitions are acceptable during the Specification phase.

## 3. Architecture Decision Enforcement

Twelve ADRs govern this project's technical direction. Flag violations of
any accepted decision.

### Storage (ADR-003)

- The storage backend is **Postgres with pgvector**. This is decided.
- Flag any import of, dependency on, or reference to: Qdrant, Pinecone,
  ChromaDB, Milvus, Weaviate, FAISS (as a primary store), or any other
  standalone vector database.
- pgvector operations must go through the storage adapter — no direct
  pgvector SQL in business logic modules.

### Embedding (ADR-007)

- Embeddings use a **swappable adapter interface** with fastembed as the
  default.
- Flag hard-coded embedding model names or dimensions outside the adapter
  layer (e.g., `model="BAAI/bge-small-en-v1.5"` in retrieval code).
- Flag direct imports of embedding libraries (fastembed, sentence-
  transformers, openai) outside `src/engram/embedding/` or wherever the
  adapter is defined.

### Agent architecture (ADR-008)

- Agent roles (RL Gate Advisor, Research Scanner, Meta-Optimizer) are
  defined using Python `Protocol` classes.
- Flag concrete class inheritance (`class Advisor(BaseAgent)`) where a
  Protocol should be used instead.
- Flag agent implementations that bypass the Librarian Gate for memory
  writes.

### Knowledge graph (ADR-009)

- The knowledge graph uses **NetworkX** for prototyping.
- Flag any introduction of Neo4j, py2neo, neomodel, or neo4j-driver as
  a dependency. Neo4j is a future migration target, not a current
  dependency.

### Retrieval (ADR-011)

- The retrieval pipeline follows the **HippoRAG 6-stage model** with
  adaptive routing.
- Flag retrieval implementations that skip stages or hard-code a single
  retrieval strategy without the routing layer.

### Librarian Gate (ADR-012)

- Memory writes go through a **hybrid gate**: declarative rules first,
  then an RL-trained advisor for edge cases.
- Flag any code that writes to memory storage directly without passing
  through the Librarian Gate.
- Flag removal or bypass of the declarative rules layer.

### Self-healing (multiple ADRs)

- Module boundaries must use **aiobreaker** circuit breakers and
  **tenacity** retries.
- Flag raw try/except blocks at module boundaries where circuit breakers
  should be used instead.
- Flag synchronous retry loops in async code.

## 4. Python Coding Standards

Based on the project's ruff and mypy configuration in `pyproject.toml`.

### Linting and formatting

- **Linter**: ruff with rule sets E, F, I, N, W, UP, B, SIM, RUF, C90,
  C4, RET, S, ARG, PLR, PLW.
- **Formatter**: ruff format, line length 88.
- **Complexity limits**: max cyclomatic complexity 10, max function args
  5, max branches 8, max returns 3, max statements 25.
- Flag functions exceeding these complexity thresholds.

### Type checking

- **mypy strict mode** is enabled. Every public function, method, and
  class attribute must have type annotations.
- Flag `Any` types in public APIs. `Any` is acceptable only in internal
  helper code with a justifying comment.
- Flag missing return type annotations on public methods.
- Flag use of `# type: ignore` without a specific error code (e.g.,
  `# type: ignore[override]` is acceptable, bare `# type: ignore` is
  not).

### Async patterns

- This is an async-first codebase. Use `async/await` throughout.
- Flag synchronous blocking calls (`time.sleep`, synchronous I/O,
  `requests.get`) in async contexts. Use `asyncio.sleep`, `aiohttp`,
  `asyncpg`, etc.
- Flag `asyncio.run()` inside library code — it should only appear in
  entry points.
- Flag `loop.run_until_complete()` — use `await` instead.

### Error handling

- Error handling belongs at **module boundaries**, not inside every
  function.
- Flag broad exception handlers: bare `except:`, `except Exception:` in
  non-boundary code.
- Flag `except` blocks that silently swallow errors (empty except, or
  just `pass`).
- Module boundary functions should use circuit breakers (aiobreaker) and
  retries (tenacity), not manual try/except chains.

### Import ordering

- Managed by ruff's isort rules: standard library, then third-party,
  then local (`engram.*`).
- Flag relative imports that reach outside the current package (e.g.,
  `from ...other_module import X`).
- Flag `import *` in any context.

### Docstrings

- Google-style docstrings for all public APIs.
- Flag public functions or classes without docstrings.
- Docstrings must document Args, Returns, and Raises sections where
  applicable.

### Module structure

- Source lives in `src/engram/` with explicit `__init__.py` exports.
- Flag modules that expose internal implementation details through
  `__init__.py` (only public interfaces should be exported).
- Flag circular imports between modules.

## 5. Security Rules

Based on STD-053 (security rules) and ruff's flake8-bandit (S) rules.

### Secrets

- **No hardcoded secrets, tokens, API keys, or passwords.** This
  includes test fixtures — use environment variables or a test secrets
  fixture.
- Flag string literals that look like secrets: API keys, JWT tokens,
  database connection strings with credentials, AWS access keys.
- Secrets must come from environment variables (via `os.environ` or
  pydantic `BaseSettings`) or a secrets manager.
- Flag `.env` files being added to version control. They must be in
  `.gitignore`.

### Input validation

- All inputs at system edges must be validated: MCP tool inputs, A2A
  messages, HTTP request bodies, query parameters.
- Flag public API endpoints or tool handlers that accept user input
  without validation (use pydantic models or explicit checks).
- Flag trust assumptions — data from external agents or users is
  untrusted until validated.

### SQL injection

- **No SQL string concatenation or f-string formatting for queries.**
- Flag any pattern matching: `f"SELECT ... {variable}"`,
  `"SELECT ... " + variable`, `cursor.execute("..." % values)`.
- All database queries must use parameterized queries or an ORM.
- pgvector similarity searches must also use parameterized queries.

### Logging

- Flag logging of secrets, tokens, full request bodies that may contain
  PII, or database connection strings.
- Sensitive fields must be redacted before logging.

## 6. Data Rules

Based on STD-051 (data rules) and ADR-003 (storage backend).

### Schema-first

- All data models must have **schema definitions before implementation
  code**. This mirrors the design-first requirement.
- Flag new data models (pydantic models, dataclasses, database tables)
  that do not reference a schema definition document.
- Memory record types (Belief, Decision, Episode, Skill, Entity,
  Preference, Reflection, Resource) have defined schemas — changes
  require a migration plan.

### Database migrations

- Migrations must be **versioned and reversible** (Alembic or
  equivalent).
- Flag migration files without a downgrade/rollback path.
- Flag DDL statements (`CREATE TABLE`, `ALTER TABLE`, `DROP`) outside
  the migration system.

### Query patterns

- No raw SQL strings in business logic. Use the storage adapter or ORM.
- pgvector operations (similarity search, index creation) must use the
  adapter pattern defined in `src/engram/` — not inline SQL.
- Flag `cursor.execute()` or `connection.execute()` with string literals
  outside the storage adapter layer.

### Memory record integrity

- Memory records are typed and versioned. Schema changes are breaking
  changes and require:
  1. A schema definition update.
  2. A versioned migration.
  3. Backward compatibility handling for existing records.
- Flag changes to memory record types that skip any of these steps.

## 7. Documentation Standards

### Markdown

- All markdown must pass **markdownlint** (config in
  `.markdownlint.yaml`).
- Use ATX-style headings (`#`, `##`, `###`), not setext or bold-as-
  heading.
- Flag `**Bold text used as a heading**` — use a proper heading level
  instead.
- Ordered lists use sequential numbering (`1.`, `2.`, `3.`), not all
  `1.`.

### Spelling

- All markdown must pass **cspell** (config in `.cspell.json`).
- Project-specific terms (engram, pgvector, HippoRAG, fastembed, etc.)
  are in the cspell word list. If a PR introduces new technical terms,
  they should be added to `.cspell.json`.

### Governed documents

- Design artifacts, specs, and governed docs require YAML frontmatter
  with: `id`, `title`, `version`, `status`, `owner`, `reviewer`,
  `approver`, `last_updated`, `extends`.
- Flag governed documents missing required frontmatter fields.
- Version numbers follow semver. Flag version decrements.

### Links

- Internal links use relative paths (not absolute GitHub URLs).
- Flag broken internal links (referencing files that do not exist in the
  PR diff or repository).

## 8. Commit and PR Conventions

### Commit messages

- **Conventional Commits** format: `type(scope): summary`
- Valid types: `feat`, `fix`, `docs`, `chore`, `refactor`, `test`, `ci`,
  `perf`.
- Valid scopes: `engram`, `design`, `architecture`, `governance`, `ci`,
  `deps`.
- Flag commit messages that do not follow this format.
- Summary must be lowercase, imperative mood, no trailing period.

### Pull request descriptions

Every PR description must include:

1. **KB standards consulted** — list of STD-XXX references relevant to
   the change.
2. **AI assistance summary** — which AI tools were used and what they
   generated.
3. **Design artifact references** — which specs, schemas, or ADRs this
   change implements or relates to.

The PR template at `.github/pull_request_template.md` defines the full
required structure. Flag PRs that omit required sections.

### Branch naming

- Format: `<type>/<short-name>` (e.g., `feature/memory-adapter`,
  `fix/retrieval-timeout`, `docs/adr-013`).
- Flag branches that do not follow this convention.

### Merge strategy

- **Squash merge only.** Flag merge commits or rebase merges.

## 9. CI Gate Awareness

Six CI gates run on PRs to `main`. Reviews should flag changes likely to
fail these gates.

| Gate | Scope | What it checks |
|------|-------|----------------|
| A | Decision tracking | `.quint/` state must reflect code changes |
| B | Documentation | markdownlint, yamllint, cspell, link-check, version-consistency |
| C | Code lint + format | ruff check, ruff format --check, mypy (Python) |
| D | Tests | pytest (Python) |
| E | Config validation | YAML and JSON schema conformance |
| F | Security | CodeQL, dependency-review (advisory) |

**Review guidance per gate:**

- **Gate A**: If the PR introduces a design decision or architecture
  change, verify that `.quint/` is updated or a Quint decision record is
  referenced.
- **Gate B**: Check that markdown changes pass lint. Watch for: missing
  blank lines around headings (MD022), bold-as-heading (MD036), trailing
  whitespace.
- **Gate C**: Verify Python code follows ruff rules. Common violations:
  unused imports (F401), unsorted imports (I001), missing type
  annotations, complexity exceeded.
- **Gate D**: Verify new code has corresponding tests. Flag untested
  public functions.
- **Gate E**: If YAML or JSON config files are modified, verify they
  conform to expected schemas.
- **Gate F**: Flag new dependencies with known vulnerabilities. Check
  `pyproject.toml` changes against pip-audit.

## 10. Module Boundaries

The system has four core modules. Reviews must verify that changes
respect these boundaries.

### Memory Module

- Typed memory records (Belief, Decision, Episode, Skill, Entity,
  Preference, Reflection, Resource).
- CRUD operations, versioning, temporal metadata.
- Owns the memory record schemas and storage adapter interfaces.

### Retrieval Module

- HippoRAG 6-stage pipeline.
- Embedding adapter integration.
- Search, reranking, context assembly.
- Depends on Memory Module for record access (through public interfaces
  only).

### Autonomy Module

- Librarian Gate (declarative rules + RL advisor).
- Agent roles (Protocol classes): RL Gate Advisor, Research Scanner,
  Meta-Optimizer.
- Depends on Memory Module for writes (through the Librarian Gate) and
  Retrieval Module for reads.

### Infrastructure Module

- Storage adapters (Postgres, pgvector).
- Protocol handlers (MCP, A2A).
- Observability (structured logging, metrics, tracing).
- Circuit breakers and retry policies.

### Boundary rules

- Cross-module imports must go through **public interfaces** defined in
  each module's `__init__.py`.
- Flag imports of internal/private symbols (prefixed with `_`) from
  other modules.
- Flag direct database access outside the Infrastructure Module's
  storage adapters.
- Flag protocol handling (MCP/A2A serialization) outside the
  Infrastructure Module.
- The dependency direction is: Infrastructure <- Memory <- Retrieval <-
  Autonomy. Flag imports that violate this direction (e.g., Memory
  importing from Retrieval).

## 11. Engineering KB Reference

This project is governed by an engineering knowledge base pinned to
**v2.0.0** of `sh4i-yurei/policies-and-standards`. The pin is tracked in
`.kb/ai-context.yaml`.

**When reviewing governance or design docs:**

- Verify that referenced standards (STD-XXX) exist in the pinned KB
  version.
- Flag governance changes that contradict the KB without an accompanying
  ADR or waiver.
- Flag changes to `.kb/ai-context.yaml` that modify the KB pin — version
  bumps require deliberate review.

**Key standards for this project:**

| Area | Standards |
|------|-----------|
| Design-first | STD-020, STD-021, STD-022, STD-023 |
| Architecture | STD-047 (ADR workflow) |
| Schema | STD-055 (schema definitions) |
| Code quality | STD-005 (coding standards), STD-008 (testing) |
| Security | STD-007 (threat modeling), STD-053 (security rules) |
| Data | STD-044 (data management), STD-051 (data rules) |
| CI/CD | STD-030 (pipeline model), STD-031 (git workflow) |
| API | STD-050 (API rules) |
| AI usage | STD-004 (AI-assisted development) |

## 12. Common Review Flags — Quick Reference

Flag these patterns immediately when spotted:

| Pattern | Reason |
|---------|--------|
| Implementation code without spec reference | STD-020 violation |
| Import of Qdrant/Pinecone/ChromaDB/Milvus/Weaviate | ADR-003 violation |
| Hard-coded embedding model name in non-adapter code | ADR-007 violation |
| Neo4j/py2neo/neomodel dependency | ADR-009 violation (premature) |
| Direct memory write bypassing Librarian Gate | ADR-012 violation |
| `f"SELECT ... {var}"` or SQL string concatenation | SQL injection risk |
| Hardcoded secret, token, or API key | STD-053 violation |
| Bare `except:` or `except Exception:` with `pass` | Silent error swallowing |
| `time.sleep()` in async code | Blocking the event loop |
| Missing type annotations on public API | mypy strict violation |
| `import *` | Namespace pollution |
| Bold text used as heading in markdown | MD036 / markdownlint violation |
| Cross-module import of private symbol (`_name`) | Module boundary violation |
| Missing docstring on public function | STD-005 violation |
| Migration without rollback path | STD-051 violation |
| PR description missing KB citations | PR template violation |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sh4i-yurei)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/sh4i-yurei)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
