---
title: Use ILIKE for Case-Insensitive Search, Not LOWER(col) LIKE
impact: HIGH
impactDescription: LOWER(col) LIKE prevents index usage — full table scan on large tables
tags: postgresql, ilike, like, search, index, case-insensitive, pg_trgm
source: The Art of PostgreSQL - Chapter 7
---

## Use ILIKE for Case-Insensitive Search, Not LOWER(col) LIKE

Wrapping a column in `LOWER()` makes the expression non-SARGable — PostgreSQL cannot use a regular B-tree index on it. `ILIKE` is natively case-insensitive and, combined with a `pg_trgm` GIN index, supports indexed pattern matching.

**Impact**: HIGH - Full table scan instead of index scan; query time grows linearly with table size

### Detection

- `LOWER(column) LIKE LOWER(?)` or `LOWER(column) LIKE ?` patterns in raw SQL
- `db.Raw("... WHERE LOWER(name) LIKE ...")` in search queries
- Absence of `CREATE INDEX ... USING GIN ... gin_trgm_ops` for ILIKE-heavy queries

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - LOWER() prevents index usage
db.Raw("SELECT * FROM users WHERE LOWER(name) LIKE LOWER(?)", "%"+query+"%").Scan(&users)
// Full table scan every time
```

### Correct (Best Practice)

```go
// ✅ GOOD - ILIKE is index-friendly with pg_trgm
db.Raw("SELECT * FROM users WHERE name ILIKE ?", "%"+query+"%").Scan(&users)

// Required: create pg_trgm GIN index in migration
// CREATE EXTENSION IF NOT EXISTS pg_trgm;
// CREATE INDEX CONCURRENTLY idx_users_name_trgm ON users USING GIN(name gin_trgm_ops);
```

### References

- *"The Art of PostgreSQL"* by Dimitri Fontaine — Chapter 7
- [PostgreSQL Docs: pg_trgm](https://www.postgresql.org/docs/16/pgtrgm.html)

---

**Rule Version**: 1.0
**Last Updated**: 2026-03-13
**Priority**: HIGH
**Auto-fixable**: Yes (LOWER(col) LIKE → ILIKE)
