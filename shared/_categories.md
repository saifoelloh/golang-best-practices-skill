# Rule Categories Index

All 40 rules organized by priority and domain.

---

## CRITICAL (12 rules) — Prevents data loss, SQL injection, crashes, production failures

| Rule | Domain | Impact |
|---|---|---|
| `critical-sql-injection` | gorm-query-patterns | SQL injection via string concatenation |
| `critical-error-check` | gorm-query-patterns | Silent failures from unchecked .Error |
| `critical-transaction-wrap` | gorm-query-patterns | Partial writes without transaction |
| `critical-placeholder-style` | postgresql-syntax | Runtime syntax error from $N in GORM context |
| `critical-n-plus-one` | query-performance | Exponential queries under load |
| `critical-unindexed-fk` | query-performance | Full table scans on every JOIN |
| `critical-errors-is-as` | error-handling | Wrong error branch from == comparison |
| `critical-pg-error-mapping` | error-handling | Postgres internals leaking to business logic |
| `critical-serialization-retry` | error-handling | Silent failures on concurrent serializable txns |
| `critical-no-automigrate-production` | migration-safety | Table locks and no rollback on every restart |
| `critical-not-null-backfill` | migration-safety | Migration fails on non-empty table |
| `critical-always-provide-rollback` | migration-safety | No recovery path on bad migration |

---

## HIGH (17 rules) — Prevents performance degradation and subtle bugs

| Rule | Domain | Impact |
|---|---|---|
| `high-preload-vs-joins` | gorm-query-patterns | N+1 or incorrect filter results |
| `high-first-vs-find` | gorm-query-patterns | Hidden ORDER BY LIMIT 1 on list queries |
| `high-save-vs-updates` | gorm-query-patterns | Silent zero-value overwrites on Save |
| `high-with-context` | gorm-query-patterns | No query timeout or cancellation |
| `high-select-columns` | gorm-query-patterns | SELECT * fetches unused large columns |
| `high-identifier-quoting` | postgresql-syntax | Reserved keyword causes syntax error |
| `high-returning-clause` | postgresql-syntax | Race condition on INSERT+SELECT pattern |
| `high-ilike-search` | postgresql-syntax | LOWER() prevents index usage |
| `high-explicit-cast` | postgresql-syntax | Type mismatch causes operator errors |
| `high-explain-analyze` | query-performance | Assumed-fast query doing Seq Scan |
| `high-connection-pool` | query-performance | Connection exhaustion under load |
| `high-find-in-batches` | query-performance | OOM on large dataset processing |
| `high-rows-affected` | error-handling | Silent no-op on Update/Delete |
| `high-wrap-with-context` | error-handling | Undebuggable errors in production |
| `high-deadlock-detection` | error-handling | 40P01 treated as fatal instead of retryable |
| `high-create-index-concurrently` | migration-safety | Write lock on large table during index build |
| `high-idempotent-migration` | migration-safety | Migration fails on re-run |

---

## MEDIUM (11 rules) — Code quality, idiomatic patterns, maintainability

| Rule | Domain | Impact |
|---|---|---|
| `medium-scopes` | gorm-query-patterns | Duplicated WHERE logic across methods |
| `medium-pagination-order` | gorm-query-patterns | Non-deterministic pagination results |
| `medium-omit-associations` | gorm-query-patterns | Unintended association upserts on Create |
| `medium-cte-readability` | postgresql-syntax | Unreadable nested subqueries |
| `medium-jsonb-operators` | postgresql-syntax | Wrong operator causes empty results on JSONB |
| `medium-interval-arithmetic` | postgresql-syntax | Error-prone manual second calculations |
| `medium-select-redundancy` | postgresql-syntax | Redundant manual quoting in .Select() |
| `medium-count-query` | query-performance | Full fetch just to call len() |
| `medium-partial-index` | query-performance | Index bloat on low-cardinality filtered queries |
| `medium-structured-logging` | error-handling | Unstructured logs not queryable in APM tools |
| `medium-drop-column-deprecation` | migration-safety | DROP COLUMN in same deploy breaks old pods |

---

## Total: 40 Rules

- **CRITICAL**: 12 rules (30%)
- **HIGH**: 17 rules (42%)
- **MEDIUM**: 11 rules (28%)

---

## How to Navigate

Rule files are in `{domain}/rules/{priority}-{name}.md`:

```
gorm-query-patterns/rules/critical-sql-injection.md
postgresql-syntax/rules/high-identifier-quoting.md
query-performance/rules/critical-n-plus-one.md
error-handling/rules/critical-pg-error-mapping.md
migration-safety/rules/critical-not-null-backfill.md
```

**Last Updated**: 2026-03-13
