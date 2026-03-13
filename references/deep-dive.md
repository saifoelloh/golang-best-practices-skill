# GORM + PostgreSQL Deep-Dive Reference

Comprehensive reference material. Load specific sections when rules in domain skills need more context.

---

## PostgreSQL Error Codes — Full Reference

| Code | Name | Retryable | Common Cause | GORM Equivalent |
|---|---|---|---|---|
| `23505` | unique_violation | No | Duplicate insert on UNIQUE column | `gorm.ErrDuplicatedKey` |
| `23503` | foreign_key_violation | No | INSERT referencing non-existent FK | — |
| `23502` | not_null_violation | No | NULL value on NOT NULL column | — |
| `23514` | check_violation | No | CHECK constraint failed | — |
| `40001` | serialization_failure | **Yes** | Concurrent transaction conflict | — |
| `40P01` | deadlock_detected | **Yes** | Deadlock between transactions | — |
| `57014` | query_canceled | No | Context timeout or `pg_cancel_backend` | — |
| `53300` | too_many_connections | No | PostgreSQL connection limit exhausted | — |
| `42P01` | undefined_table | No | Table doesn't exist (missing migration) | — |
| `42703` | undefined_column | No | Column doesn't exist (missing migration) | — |
| `22001` | string_data_right_truncation | No | String value too long for column | — |
| `22P02` | invalid_text_representation | No | Invalid UUID format or bad type cast | — |

**How to extract the code in Go:**

```go
import "github.com/jackc/pgx/v5/pgconn"

var pgErr *pgconn.PgError
if errors.As(err, &pgErr) {
    fmt.Println(pgErr.Code)    // "23505"
    fmt.Println(pgErr.Detail)  // "Key (email)=(test@test.com) already exists."
    fmt.Println(pgErr.ConstraintName)  // "users_email_key"
    fmt.Println(pgErr.ColumnName)      // available for not_null_violation
}
```

---

## GORM Method Cheat Sheet

| Symptom | Root Cause | Rule | Fix |
|---|---|---|---|
| Empty result, no error | Missing `.Error` check | `critical-error-check` | `if err := db.xxx().Error; err != nil` |
| Extra `ORDER BY id` in list query | Used `db.First` on slice | `high-first-vs-find` | Use `db.Find` for lists |
| All fields reset to zero on update | Used `db.Save` with partial struct | `high-save-vs-updates` | Use `db.Updates(map[...])` |
| Missing association data | No `Preload` | `high-preload-vs-joins` | Add `Preload("AssociationName")` |
| N+1 queries in loop | Manual DB call inside range loop | `critical-n-plus-one` | Use `Preload` before loop |
| Pagination non-deterministic | Missing `Order` | `medium-pagination-order` | Add `.Order("created_at DESC, id DESC")` |
| Context not propagated | Missing `WithContext` | `high-with-context` | Add `db.WithContext(ctx)` |
| Association upserted unexpectedly | GORM default upsert on Create | `medium-omit-associations` | Add `.Omit(clause.Associations)` |
| `operator does not exist: uuid = text` | Missing type cast | `high-explicit-cast` | Use `?::uuid` placeholder |
| `syntax error at or near "order"` | Unquoted reserved keyword | `high-identifier-quoting` | Use `"order"` (double-quoted) |
| `syntax error at or near "$1"` | Wrong placeholder style in GORM | `critical-placeholder-style` | Use `?` not `$1` |

---

## Index Design Guide

### Rule of Thumb: Index What You Filter On

```sql
-- Single column equality
CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_users_email ON users(email);

-- Multi-column: most selective column first, then the range/filter column
CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_orders_user_status ON orders(user_id, status);

-- Partial index: only the rows you actually query (e.g., pending only)
CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_orders_pending ON orders(created_at)
    WHERE status = 'pending';

-- GIN index for JSONB containment (@>)
CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_orders_metadata ON orders USING GIN(metadata);

-- GIN index for ILIKE search (requires pg_trgm extension)
CREATE EXTENSION IF NOT EXISTS pg_trgm;
CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_users_name_trgm ON users USING GIN(name gin_trgm_ops);
```

### When NOT to Add Indexes

- Columns with very few distinct values (< ~10) — use partial index instead
- Tables with < ~1,000 rows — sequential scan is faster
- Write-only columns never used in WHERE/JOIN/ORDER

### Verify Index Usage

```sql
-- Check if query uses your index
EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM orders WHERE user_id = 1 AND status = 'pending';
-- Look for: "Index Scan using idx_orders_user_status"
-- Avoid: "Seq Scan on orders" (missing index)

-- Check index usage stats
SELECT schemaname, tablename, indexname, idx_scan, idx_tup_read
FROM pg_stat_user_indexes
WHERE tablename = 'orders'
ORDER BY idx_scan DESC;
```

---

## Zero-Downtime Migration Checklist

Before running any migration in production:

- [ ] Does migration add a NOT NULL column? → Need DEFAULT or 3-step backfill (see `critical-not-null-backfill`)
- [ ] Does migration create an index? → Use `CONCURRENTLY` (see `high-create-index-concurrently`)
- [ ] Does migration drop a column? → Code references removed in prior deploy? (see `medium-drop-column-deprecation`)
- [ ] Does migration have a corresponding `down.sql`? (see `critical-always-provide-rollback`)
- [ ] Is migration idempotent (`IF EXISTS` / `IF NOT EXISTS`)? (see `high-idempotent-migration`)
- [ ] Tested on a copy of production data size?
- [ ] Is `CONCURRENTLY` index outside a transaction block?

---

## Connection Pool Tuning Guide

```go
sqlDB.SetMaxOpenConns(25)                   // Start at 20-30
sqlDB.SetMaxIdleConns(10)                   // 30-50% of MaxOpenConns
sqlDB.SetConnMaxLifetime(5 * time.Minute)   // Prevent stale SSL sessions
sqlDB.SetConnMaxIdleTime(1 * time.Minute)   // Release during low-traffic
```

**Sizing Rule**: `MaxOpenConns` across all app instances must be ≤ PostgreSQL `max_connections - 10` (reserve for admin connections).

Check current PostgreSQL `max_connections`:
```sql
SHOW max_connections;  -- Default: 100
```

---

## Authoritative Sources

| Topic | Source | URL |
|---|---|---|
| GORM queries | GORM Official Docs | https://gorm.io/docs/query.html |
| GORM associations | GORM Official Docs | https://gorm.io/docs/has_many.html |
| GORM error handling | GORM Official Docs | https://gorm.io/docs/error_handling.html |
| PostgreSQL indexes | Use The Index, Luke | https://use-the-index-luke.com |
| PostgreSQL error codes | PG Docs Appendix A | https://www.postgresql.org/docs/16/errcodes-appendix.html |
| EXPLAIN ANALYZE | PG Docs | https://www.postgresql.org/docs/16/sql-explain.html |
| Zero-downtime migrations | Braintree Engineering | https://www.braintreepayments.com/blog/safe-operations-for-high-volume-postgresql |
| Postgres SQL idioms | The Art of PostgreSQL | https://theartofpostgresql.com |
| pgx error types | pgx GoDoc | https://pkg.go.dev/github.com/jackc/pgx/v5/pgconn |
