---
name: golang-fullstack-error-handling
description: Unified Go + GORM + PostgreSQL error handling review. Covers error wrapping, context propagation, PostgreSQL error codes (23505, 40001), errors.Is/As mapping, and retry logic. Ensures proper error chains and robust database failure recovery.
license: MIT
metadata:
  author: saifoelloh
  version: "3.0.0"
  parent_skill: golang-fullstack-best-practices
  sources:
    - "Learning Go: An Idiomatic Approach (Jon Bodner)"
    - "PostgreSQL Error Codes Appendix"
    - "GORM Official Documentation"
    - "Go Blog - Working with Errors"
  last_updated: "2026-03-13"
---

# Error Handling (Unified Go & DB)

Expert-level review for error handling in Go services using GORM and PostgreSQL. Ensures errors are correctly identified, preserved via wrapping, mapped to domain errors, and retried where appropriate.

## When to Apply

Use this skill when:
- Reviewing general Go error propagation and context usage
- Debugging database errors (e.g., duplicate keys, serialization failures)
- Auditing error wrapping with `%w` vs `%v`
- Ensuring correct use of `errors.Is` / `errors.As`
- Adding retry logic for serializable transactions or deadlocks

## Rule Categories

| Priority | Count | Focus |
|---|---|---|
| **CRITICAL** | 6 | error chains, context leaks, PgError mapping, retries |
| **HIGH** | 6 | propagation, Is/As usage, RowsAffected, deadlock detection |
| **MEDIUM** | 2 | sentinel errors, structured logging |

## Rules Covered (14 total)

### Critical Issues (6)
- `critical-error-wrapping` — Use `%w`, not `%v`, to preserve error chains
- `critical-context-leak` — Always `defer cancel()` after context creation
- `critical-error-shadow` — Don't shadow `err` in nested scopes
- `critical-errors-is-as` — Use `errors.Is`/`As` for DB and wrapped errors
- `critical-pg-error-mapping` — Map Postgres codes to domain errors at repo boundary
- `critical-serialization-retry` — Retry transactions on `40001` serialization failure

### High-Impact Patterns (6)
- `high-context-propagation` — Propagate context through the call chain
- `high-error-is-as` — Prescriptive rule for using Is/As over string matching
- `high-interface-nil` — Check for nil in interfaces correctly
- `high-rows-affected` — Check `RowsAffected` after Update/Delete
- `high-wrap-with-context` — Wrap DB errors with operation context
- `high-deadlock-detection` — Handle `40P01` deadlocks as retryable

### Medium Improvements (2)
- `medium-sentinel-error-usage` — Use sentinel errors for stable categories
- `medium-structured-logging` — Log DB/Go errors with structured fields

## Trigger Phrases

- "Review error handling"
- "Debug this DB error"
- "23505 / 40001 / 40P01"
- "Check for context leaks"
- "Review error chains"
- "Transaction retry"
- "ErrRecordNotFound"

## Output Format

```
## Critical Issues Found: X

### [Rule ID] (Line Y)
**Issue**: Description
**Impact**: Context leak / Lost chain / Data inconsistency
**Fix**: Suggested code
**Example**:
```go
// Corrected code
```
```

## Related Skills
- [gorm-query-patterns](../gorm-query-patterns/SKILL.md) — For `.Error` check safety
- [concurrency-safety](../concurrency-safety/SKILL.md) — For context timeout patterns
