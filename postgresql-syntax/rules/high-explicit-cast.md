---
title: Cast Types Explicitly — Don't Rely on Implicit Coercion
impact: HIGH
impactDescription: Implicit type mismatch causes "operator does not exist" errors or index misses
tags: postgresql, type-casting, uuid, bigint, explicit-cast, operator
source: PostgreSQL Docs - Type Casting
---

## Cast Types Explicitly — Don't Rely on Implicit Coercion

PostgreSQL is strongly typed. Comparing a `UUID` column with a plain `string` parameter, or a `BIGINT` with an `INT` without explicit casting, can trigger `"operator does not exist: uuid = text"` errors, or silently bypass indexes.

**Impact**: HIGH - Runtime `operator does not exist` errors; index bypass on type mismatch

### Detection

- UUID columns queried with untyped string parameters in raw SQL
- `WHERE id = ?` where `id` is `UUID` type but Go passes a `string`
- Absence of `::uuid`, `::bigint`, `::text` casts on ambiguous parameters

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - UUID column vs plain string → "operator does not exist: uuid = text"
db.Raw("SELECT * FROM users WHERE id = ?", "123e4567-e89b-12d3-a456-426614174000").Scan(&user)
```

### Correct (Best Practice)

```go
// ✅ GOOD - explicit cast in SQL
db.Raw("SELECT * FROM users WHERE id = ?::uuid", "123e4567-e89b-12d3-a456-426614174000").Scan(&user)

// ✅ BEST - use typed UUID in Go struct (GORM handles it)
import "github.com/google/uuid"

type User struct {
    ID uuid.UUID `gorm:"type:uuid;default:gen_random_uuid()"`
}
db.First(&user, uuid.MustParse("123e4567-e89b-12d3-a456-426614174000"))
```

### References

- [PostgreSQL Docs: Type Casting](https://www.postgresql.org/docs/16/sql-expressions.html#SQL-SYNTAX-TYPE-CASTS)

---

**Rule Version**: 1.0
**Last Updated**: 2026-03-13
**Priority**: HIGH
**Auto-fixable**: No
