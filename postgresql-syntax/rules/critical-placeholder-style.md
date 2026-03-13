---
title: Use ? Placeholders with GORM, $N Only with pgx/pq Directly
impact: CRITICAL
impactDescription: Mixing placeholder styles causes runtime syntax error — query silently fails in production
tags: postgresql, gorm, placeholder, raw-sql, pgx, pq, syntax-error
source: GORM Docs - Raw SQL, PostgreSQL Protocol Docs
---

## Use ? Placeholders with GORM, $N Only with pgx/pq Directly

GORM uses `?` as its placeholder syntax and internally converts it to PostgreSQL's `$1, $2, ...` positional parameters. Using `$1` directly in `db.Raw()` bypasses GORM's conversion and causes a runtime error.

**Impact**: CRITICAL - `ERROR: syntax error at or near "$1"` at runtime, query completely fails

### Detection

How to spot this issue:
- `db.Raw("SELECT * FROM users WHERE id = $1", id)` — `$N` in GORM context
- `db.Where("status = $1", status)` — `$N` in GORM Where clause
- Mixing `$N` and `?` in the same GORM query string

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - $1 syntax doesn't work with GORM's db.Raw()
func (r *UserRepository) FindByID(id uint) (*User, error) {
    var user User
    err := r.db.Raw("SELECT * FROM users WHERE id = $1", id).Scan(&user).Error
    // Runtime error: pq: syntax error at or near "$1"
    return &user, err
}

// ❌ BAD - also wrong in Where
r.db.Where("tenant_id = $1 AND status = $2", tenantID, "active").Find(&users)
```

**Why this is wrong**:
- GORM does not pass raw SQL strings directly to PostgreSQL — it processes `?` into `$N` via its dialects
- Passing `$1` directly confuses GORM's parameter binder, resulting in a syntax error

### Correct (Best Practice)

```go
// ✅ CORRECT - use ? with GORM
func (r *UserRepository) FindByID(ctx context.Context, id uint) (*User, error) {
    var user User
    if err := r.db.WithContext(ctx).Raw("SELECT * FROM users WHERE id = ?", id).
        Scan(&user).Error; err != nil {
        return nil, fmt.Errorf("UserRepository.FindByID: %w", err)
    }
    return &user, nil
}

// ✅ CORRECT - $N is correct when using pgx or pq directly (without GORM)
func QueryDirect(pool *pgxpool.Pool, ctx context.Context, id uint) (*User, error) {
    row := pool.QueryRow(ctx, "SELECT id, name FROM users WHERE id = $1", id)
    var user User
    err := row.Scan(&user.ID, &user.Name)
    return &user, err
}

// ✅ CORRECT - GORM named args (alternative to ?)
r.db.Raw("SELECT * FROM users WHERE id = @id AND status = @status",
    sql.Named("id", id),
    sql.Named("status", "active"),
).Scan(&user)
```

**Why this is better**:
- `?` is the consistent placeholder for all GORM operations (GORM handles dialect conversion)
- Named args with `sql.Named` are readable in complex queries
- `$N` is reserved for direct `pgx`/`pq` usage outside GORM

### References

- [GORM Docs: Raw SQL and SQL Builder](https://gorm.io/docs/sql_builder.html)
- [PostgreSQL Protocol: Message Formats](https://www.postgresql.org/docs/16/protocol-message-formats.html)

---

**Rule Version**: 1.0
**Last Updated**: 2026-03-13
**Priority**: CRITICAL
**Auto-fixable**: Yes (replace $N with ? in GORM context)
