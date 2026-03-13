---
title: Use errors.Is and errors.As to Check GORM and PostgreSQL Errors
impact: CRITICAL
impactDescription: Direct == comparison fails on wrapped errors — wrong error handling branch taken
tags: gorm, error-handling, errors-is, errors-as, error-wrapping, pgconn
source: GORM Docs - Error Handling, Go Blog - Working with Errors
---

## Use errors.Is and errors.As to Check GORM and PostgreSQL Errors

GORM wraps errors through multiple layers. Using `==` to compare errors (e.g., `err == gorm.ErrRecordNotFound`) fails when the error has been wrapped with `fmt.Errorf(...%w...)`. Always use `errors.Is` for sentinel errors and `errors.As` for type assertions.

**Impact**: CRITICAL - Wrong error handling branch taken silently — "not found" treated as generic error or vice versa

### Detection

- `err == gorm.ErrRecordNotFound` — direct equality on GORM sentinel error
- `strings.Contains(err.Error(), "record not found")` — brittle string matching
- No `errors.As(err, &pgErr)` for PostgreSQL-specific error code handling

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - direct == fails if error is wrapped
if err == gorm.ErrRecordNotFound {
    return nil, ErrUserNotFound
}

// ❌ BAD - string matching is brittle and locale-dependent
if strings.Contains(err.Error(), "record not found") {
    return nil, ErrUserNotFound
}
```

### Correct (Best Practice)

```go
import (
    "errors"
    "github.com/jackc/pgx/v5/pgconn"
    "gorm.io/gorm"
)

// ✅ GOOD - errors.Is unwraps the error chain correctly
func mapDBError(err error) error {
    if err == nil {
        return nil
    }
    // GORM sentinel errors
    if errors.Is(err, gorm.ErrRecordNotFound) {
        return ErrNotFound
    }
    if errors.Is(err, gorm.ErrDuplicatedKey) {
        return ErrAlreadyExists
    }
    // PostgreSQL error codes via errors.As
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
    return fmt.Errorf("database error: %w", err)
}
```

### References

- [GORM Docs: Error Handling](https://gorm.io/docs/error_handling.html)
- [Go Blog: Working with Errors](https://go.dev/blog/go1.13-errors)

---

**Rule Version**: 1.0
**Last Updated**: 2026-03-13
**Priority**: CRITICAL
**Auto-fixable**: Yes (== → errors.Is)
