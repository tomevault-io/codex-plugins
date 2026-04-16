# GitHub Copilot – Code Review Instructions

## Review Philosophy

1. Only comment when you have **HIGH CONFIDENCE (>80%)** that an issue exists.
2. Avoid speculation. Never say _"consider changing…"_ unless it's a real concern.
3. Be concise. One sentence per comment when possible.
4. Every comment should be **actionable**, not observational.
5. When reviewing text or comments, only flag issues if clarity impacts understanding or functionality.
6. Assume the contributor made intentional choices unless there's evidence otherwise.

---

## Response Format

1. **State the problem clearly.** (1 sentence)
2. **Explain why it matters.** (Optional)
3. **Suggest a fix.** (Code snippet or concrete action)

### Example

Avoid using `unwrap()` here; it could panic if `None`.  
Use `?` or handle the error explicitly with `if let Some(val) = ...`.

---

## When to Stay Silent

1. If you're not sure something is a problem, say nothing.
2. Avoid commenting just to _"say something."_
3. Don't suggest alternatives unless the current approach has real risks.

---

## Spring Boot Application

This document defines **mandatory code review rules** for GitHub Copilot when reviewing pull requests in this repository.  
All rules are **non-negotiable** unless explicitly approved by the CTO.

---

## 1. Database Query & SQL Standards (Mandatory)

### 1.1 WHERE Clause Restrictions

- ❌ **Do NOT add new fields** in the `WHERE` clause of `SELECT` queries.
- ✅ If additional filtering is required:
  - Fetch the base dataset.
  - Apply filtering **using object iteration in Java** (Streams, loops, predicates).
- Copilot must flag any PR that adds new `WHERE` conditions to existing queries.

---

### 1.2 Prohibition of `SELECT *`

- ❌ `SELECT *` is **strictly prohibited**.
- ✅ Explicitly specify required columns in all queries.
- Copilot must suggest replacing `SELECT *` with named columns.

---

### 1.3 Mandatory Use of SQL Aliases

- ✅ All tables and columns **must use aliases**.
- ❌ Queries without aliases must be flagged.

**Example:**

```sql
SELECT u.id AS userId, u.name AS userName
FROM users u
```
---

### 1.4 Pagination & COUNT(*) Usage – Strictly Prohibited

Pageable and Page<T> Are Not Allowed:
- ❌ Usage of Pageable, Page<T>, or Spring Data auto-pagination is strictly prohibited.
- ❌ Repository methods must not accept Pageable as a parameter.
- ❌ Repository methods must not return Page<T>.
- ✅ use the slice interface instead of Pageable
```
void processAll() {
    Slice<Employee> slice = repository.findAllBy(PageRequest.of(0, BATCH_SIZE));
    List<Employee> employees = slice.getContent();
    employees.forEach(this::doSomething);
 
    while(slice.hasNext()) {
        slice = repository.findAllBy(slice.nextPageable());
        slice
            .getContent()
            .forEach(this::doSomething);
    }
}
```

---

### 1.5 DTO-Based Query Result Mapping (Mandatory)
- ❌ Do NOT access query results using positional indexes (e.g., row[0], row[1]).
- ❌ Avoid Object[]–based result processing in service or DAO layers.
- ✅ All multi-column query results must be mapped to a DTO.
- ✅ Queries must use explicit column aliases that match DTO field or constructor parameter names.
- ✅ Prefer constructor-based projections for immutability and clarity.
Example:
// ❌ WRONG
Object[] row = (Object[]) result;
Long citizenId = (Long) row[0];

//  ✅ CORRECT:
```
public class EpisodeMappingDTO {
    public EpisodeMappingDTO(Long citizenId, Long episodeTypeId, Long modId, String idsString) {}
}
-- JPQL / Native Query
SELECT
    c.id AS citizenId,
    e.typeId AS episodeTypeId,
    m.id AS modId,
    x.ids AS idsString
FROM ...
// Result mapping
EpisodeMappingDTO dto = (EpisodeMappingDTO) result;
```

---

### 1.6 Index Mapping Validation for Query Changes (Mandatory)

- ✅ Any code change that **adds, modifies, or introduces a SQL query** must be validated against existing indexes.
- ✅ The reviewer (Copilot) **must scan the SQL files under `table_schema/`** to identify:
  - Primary keys
  - Unique indexes
  - Composite indexes
  - Foreign key–backed indexes
- ❌ Queries that introduce **new filter, join, or order-by columns** without a supporting index must be flagged.

### What Copilot Must Check
- WHERE clause columns are covered by an index (leading columns for composite indexes).
- JOIN conditions use indexed columns on at least one side (preferably both).
- ORDER BY / GROUP BY columns are index-backed when used on large tables.
- Existing indexes are reused and not bypassed due to function usage or type mismatch.

### Mandatory Reviewer Action
- If an index exists:  
  - Confirm the query aligns with the index column order.
- If no suitable index exists:  
  - Flag the PR and request an **explicit justification** or an **index addition via schema change**.

### Example
**Query Change:**
```sql
SELECT *
FROM patient_visit pv
WHERE pv.citizen_id = :citizenId
  AND pv.visit_date >= :fromDate;
Schema Scan (table_schema/patient_visit.sql): 
```
---

## 2. Exception Handling

### 2.1 No Empty Catch Blocks in controller

❌ Empty catch blocks are strictly prohibited.
✅ Always log exceptions with context or rethrow them.

###2.2 Mandatory Controller-Level Exception Handling (Strict)

- ❌ Controllers must NOT expose unhandled exceptions.
- ❌ Controllers must NOT rely on default exception propagation.
- ✅ Every controller method must be wrapped in a try-catch block.
- ✅ Logging must include stack trace context using Thread.currentThread().getStackTrace().
- ✅ Errors must return a standardized ResponseDto, not raw exceptions.
- ✅ Even error responses must return ResponseEntity.ok() with an internal response code.
Mandatory Pattern (Must Be Followed Exactly)
```
protected ResponseEntity<ResponseDto> getEpisode(@RequestParam Long citizenID) {
    StackTraceElement[] stackTraceElements = Thread.currentThread().getStackTrace();

    try {

    } catch (Exception e) {
        log.error(Constant.LOGGER_PLACE_HOLDER_3,
                loggerUtil.getLogLocation(getClass().getName(), stackTraceElements), e);
        return ResponseEntity.ok(ResponseDto.builder()
                .responseCode(HttpStatus.INTERNAL_SERVER_ERROR.value())
                .responseMessage(e.getMessage())
                .build());
    }
}
```

## 3. Test case coverage

### 2.1 Test case coverage and assertion

- ✅ All modified and new methods added must have more than 80%  test case coverage.
- ✅ Ensure all the possible scenerios with assert is added without fail.
- ✅ Test case execution time must not exceed 50ms.



```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Artem-Health-Tech-Pvt-Ltd) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
