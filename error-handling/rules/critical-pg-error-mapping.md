---
title: Map PostgreSQL Error Codes to Domain Errors at the Repository Layer
impact: CRITICAL
impactDescription: Raw pgconn.PgError leaking to callers creates tight coupling and exposes DB internals
tags: gorm, postgresql, error-codes, domain-errors, pgconn, 23505, 23503
source: PostgreSQL Error Codes Appendix
---

## Map PostgreSQL Error Codes to Domain Errors at the Repository Layer

Raw `*pgconn.PgError` values (with Postgres error codes like `23505`) must be translated into meaningful domain errors at the repository boundary. Callers (usecases, handlers) should never need to know Postgres error codes.

**Impact**: CRITICAL - Postgres internals leak into business logic; callers can't handle errors meaningfully

### PostgreSQL Error Codes Quick Reference

| Code | Name | Common Cause |
|---|---|---|
| `23505` | unique_violation | Duplicate insert on UNIQUE column |
| `23503` | foreign_key_violation | INSERT referencing non-existent FK |
| `23502` | not_null_violation | NULL value on NOT NULL column |
| `23514` | check_violation | CHECK constraint failed |
| `40001` | serialization_failure | Concurrent txn conflict (retryable) |
| `40P01` | deadlock_detected | Deadlock between transactions |
| `57014` | query_canceled | Context timeout or `pg_cancel_backend` |
| `53300` | too_many_connections | PostgreSQL connection limit hit |

### Correct (Best Practice)

```go
// ✅ GOOD - repository translates PgError to domain errors
func (r *UserRepository) Create(ctx context.Context, user *User) error {
    if err := r.db.WithContext(ctx).Create(user).Error; err != nil {
        return mapDBError(err)
    }
    return nil
}

func mapDBError(err error) error {
    if errors.Is(err, gorm.ErrRecordNotFound) {
        return ErrNotFound
    }
    if errors.Is(err, gorm.ErrDuplicatedKey) {
        return ErrAlreadyExists
    }
    var pgErr *pgconn.PgError
    if errors.As(err, &pgErr) {
        switch pgErr.Code {
        case "23505":
            return fmt.Errorf("%w: %s", ErrDuplicateEntry, pgErr.Detail)
        case "23503":
            return fmt.Errorf("%w: %s", ErrInvalidReference, pgErr.ConstraintName)
        case "23502":
            return fmt.Errorf("%w: column %s", ErrMissingRequired, pgErr.ColumnName)
        case "40001":
            return ErrRetryable
        case "57014":
            return ErrQueryTimeout
        }
    }
    return fmt.Errorf("database: %w", err)
}

// Caller (usecase) handles clean domain errors — no Postgres knowledge needed
func (uc *UserUseCase) Register(ctx context.Context, req RegisterRequest) error {
    err := uc.userRepo.Create(ctx, &User{Email: req.Email})
    if errors.Is(err, ErrDuplicateEntry) {
        return ErrEmailAlreadyRegistered
    }
    return err
}
```

### References

- [PostgreSQL Error Codes Appendix](https://www.postgresql.org/docs/16/errcodes-appendix.html)
- [pgx pgconn.PgError](https://pkg.go.dev/github.com/jackc/pgx/v5/pgconn#PgError)

---

**Rule Version**: 1.0
**Last Updated**: 2026-03-13
**Priority**: CRITICAL
**Auto-fixable**: No
