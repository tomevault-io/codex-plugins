# Copilot Instructions - Universal Principles

These instructions apply to all interactions regardless of technology stack or project type.

---

## Engineering Principles (Priority Order)

1. **Correctness** - Code must work as intended
2. **Readability** - Code must be understandable by others
3. **Maintainability** - Code must be easy to modify safely
4. **Longevity** - Code must age well as dependencies evolve
5. **Performance** - Optimize only after the above are satisfied

---

## Anti-Hallucination Policy

**Hallucination is unacceptable.** Never make assumptions about code behavior, APIs, or library features without verification.

### Rules

- When asked how something works, **trace the actual code** before answering
- If documentation or code is not visible, **say so explicitly** and request it
- Never fill knowledge gaps with assumptions - state what you know and what you need
- When suggesting libraries or APIs, only reference features you can verify exist
- If uncertain about syntax or behavior, look it up or ask for clarification

### Examples

**Bad:**
> "This library probably has a method called `fetchAll()` that does what you need."

**Good:**
> "I don't have the library documentation in context. Can you share it, or should I search for the current API?"

**Bad:**
> "The latest version of React is 18.2 and supports..."

**Good:**
> "Let me verify the current React version and its features before proceeding."

---

## Code Quality Standards

### Declarative Over Imperative

Prefer code that describes **what** should happen, not **how** to do it step-by-step.

```
# Imperative (avoid)
result = []
for item in items:
    if item.is_valid:
        result.append(item.value)

# Declarative (prefer)
result = [item.value for item in items if item.is_valid]
```

### Modern Practices

- Use the latest stable language features appropriate for the target version
- Prefer built-in language features over external libraries when equivalent
- Use type systems where available (TypeScript, Python type hints, etc.)
- Prefer immutability and pure functions where practical

### Code Volume

Less code is generally better. Every line is a maintenance burden.

- Use standard library functions instead of reimplementing
- Extract reusable helpers instead of copy-pasting
- Use language idioms (comprehensions, destructuring, etc.)
- If code needs extensive comments to explain, refactor for clarity

### Naming

- Names should be descriptive and self-documenting
- Never use single-letter variables except `i`, `j`, `k` for loop indices
- Boolean variables/functions should read as yes/no questions (`is_valid`, `has_permission`)
- Functions should describe their action (`calculate_total`, `fetch_user`, `validate_input`)
- **Language/framework-specific conventions** (casing, prefixes, etc.) are defined in the relevant agent instructions

---

## Debugging Philosophy

**Fix root causes, not symptoms.**

When encountering bugs:

1. **Identify the root cause** - trace back to where the invariant was violated
2. **Fix the root cause** - correct initialization, validation, or control flow
3. **Add defensive checks** - only as diagnostics, not as the fix

### Example

**Bad (hiding the problem):**
```
if data is None:
    return default_value  # Silently hide the bug
```

**Good (fix + diagnose):**
```
# Root cause: data source not initialized before use
# Fix: ensure initialization in setup()
# Defensive check for diagnosis:
if data is None:
    logger.error("data was None - initialization may have failed")
    raise ValueError("data required but not initialized")
```

---

## Comments and Documentation

### When to Comment

- Explain **why** something is done, not **what** (the code shows what)
- Document non-obvious side effects or dependencies
- Explain business logic that isn't apparent from code
- Reference external systems, tickets, or decisions

### When NOT to Comment

- Don't restate what the code obviously does
- Don't leave commented-out code
- Don't use comments to patch unclear code - refactor instead

### Style

- Comments start with a capital letter and end with a period.
- Comments go above the code they describe, not beside it.
- Keep comments concise.

---

## Code Flow and Readability

Code should read like prose: "First we do this, then we do that, then we do this other thing."

### Single Responsibility Per Block

Each loop or block should do one clear thing. When validation, transformation, and side effects are interleaved, the code becomes hard to reason about, test, and modify.

```
# Bad: validation, transformation, and side effects interleaved
# Reader must track three concerns simultaneously; bugs hide easily.
results = []
for item in items:
    if item.price < 0:
        log.warning(f"Invalid price for {item.id}")
        continue
    adjusted = item.price * tax_rate
    if adjusted > threshold:
        send_alert(item)
    results.append(Item(item.id, adjusted))

# Good: separate passes with clear purpose
# Each step does one thing; easy to test, modify, or reorder.
valid_items = [item for item in items if validate_price(item)]
adjusted_items = [apply_tax(item, tax_rate) for item in valid_items]
alert_high_value(adjusted_items, threshold)
```

### Guard Clauses and Early Returns

Handle error cases and edge cases first, then proceed with the main logic. This keeps code flat and readable.

```
# Bad: deeply nested logic
def process_user(user):
    if user:
        if user.is_valid():
            if user.has_permission():
                # actual work buried here
                do_work(user)
                return True
    return False

# Good: guard clauses, flat structure
def process_user(user):
    if not user:
        return False
    if not user.is_valid():
        return False
    if not user.has_permission():
        return False

    do_work(user)
    return True
```

### Reduce Entropy

Minimize the cognitive load on readers. Every variable or resource that's "live" forces the reader to remember it while reading unrelated code. Group related operations together and scope resources tightly.

```
# Bad: reader must track 'connection' across 4 unrelated lines
# "Wait, when does connection get used? Is it still valid here?"
connection = db.connect()
user = parse_user_input(request)
validate_permissions(user)
config = load_feature_flags()
apply_rate_limits(user, config)
result = connection.execute(build_query(user))  # finally!

# Good: group by concern, minimize active mental state
# Reader only tracks what's relevant to the current block.
user = parse_user_input(request)
validate_permissions(user)

config = load_feature_flags()
apply_rate_limits(user, config)

with db.connect() as connection:  # scoped, used immediately
    result = connection.execute(build_query(user))
```

---

## Testing Philosophy

### Behavior Over Implementation

- Tests should verify **outcomes**, not **how** code achieves them
- Each test should answer: "What bug would this catch?"
- If you can break the implementation without failing the test, the test is too narrow

### What to Test (Priority Order)

1. Error handling and failure modes
2. Edge cases and boundary conditions
3. Core business logic correctness
4. Integration points between modules
5. Regression prevention for known bugs

### What NOT to Test

- Trivial getters/setters
- Framework or library behavior
- Implementation details that could change
- Mock objects themselves

### Test Quality

- Tests should be deterministic (no flaky tests)
- Tests should be independent (no shared mutable state)
- Tests should be fast (mock I/O at boundaries)
- Tests should be readable (clear arrange-act-assert structure)

---

## Error Handling

- Fail fast with meaningful error messages
- Include context in errors (what was expected vs. actual)
- Use appropriate error types (user error vs. system error)
- Log enough information to diagnose issues without exposing secrets

---

## Version Control

### Commit Messages

```
<scope>: <concise change description>

- Why: <reason or goal>
- What: <key changes>
- Test: <how to verify>
```

### Commits Should Be

- Atomic (one logical change per commit)
- Reversible (can be reverted independently)
- Descriptive (future you should understand why)

---

## Security Basics

- Never execute untrusted input
- Never log secrets or credentials
- Validate and sanitize all external input
- Use parameterized queries for databases
- Keep dependencies updated and minimal

---

## When Researching or Learning

- Verify information is current (check dates on documentation)
- Prefer official documentation over blog posts
- Cross-reference multiple sources for important decisions
- When in doubt, test in isolation before integrating


## When external code like libraries or packages are involved
- If you are not 100% sure of the version being used after looking, ask the user.
- All code and suggestions and research should consider the correct version of the library/package in question, otherwise
  API calls that don't exist, hallucinations, or incorrect understandings are more likely to occur. 
- Never make assumptions about what the library does if how it works is important. Ask the user to provide source code
  if the answer is not in documentation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lifeforce-dev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
