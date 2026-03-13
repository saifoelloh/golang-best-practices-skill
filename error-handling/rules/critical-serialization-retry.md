---
title: Retry Transactions on serialization_failure (40001)
impact: CRITICAL
impactDescription: Serializable transactions fail under concurrency — unretried failures cause silent data loss
tags: gorm, transaction, serialization, 40001, retry, concurrency
source: PostgreSQL Docs - Transaction Isolation
---

## Retry Transactions on serialization_failure (40001)

PostgreSQL's `SERIALIZABLE` isolation level (and to a lesser extent `REPEATABLE READ`) may abort a transaction with `ERROR 40001: could not serialize access due to concurrent update`. This is expected behavior — the correct response is to **retry** the transaction, not treat it as a hard failure.

**Impact**: CRITICAL - Unretried serialization failures cause user-visible errors or silent data loss under concurrent load

### Detection

- Transactions using `SERIALIZABLE` or `REPEATABLE READ` isolation without retry logic
- `40001` error reaching the usecase or handler layer as a generic error

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - no retry, user gets an error on concurrent transaction
func (uc *OrderUseCase) PlaceOrder(ctx context.Context, req PlaceOrderRequest) error {
    return uc.db.Transaction(func(tx *gorm.DB) error {
        // ... business logic
        return nil
    })
    // Under concurrent load: random "serialization failure" errors to users
}
```

### Correct (Best Practice)

```go
// ✅ GOOD - retry up to 3 times on 40001
func WithSerializableRetry(db *gorm.DB, maxRetries int, fn func(*gorm.DB) error) error {
    for attempt := 1; attempt <= maxRetries; attempt++ {
        err := db.Transaction(fn)
        if err == nil {
            return nil
        }
        var pgErr *pgconn.PgError
        if errors.As(err, &pgErr) && pgErr.Code == "40001" {
            if attempt < maxRetries {
                // Exponential backoff: 50ms, 100ms, 200ms
                time.Sleep(time.Duration(attempt) * 50 * time.Millisecond)
                continue
            }
        }
        return err // Non-retryable error or max retries exceeded
    }
    return ErrMaxRetriesExceeded
}

// Usage
func (uc *OrderUseCase) PlaceOrder(ctx context.Context, req PlaceOrderRequest) error {
    return WithSerializableRetry(uc.db.WithContext(ctx), 3, func(tx *gorm.DB) error {
        // ... business logic
        return nil
    })
}
```

### References

- [PostgreSQL Docs: Transaction Isolation](https://www.postgresql.org/docs/16/transaction-iso.html)

---

**Rule Version**: 1.0
**Last Updated**: 2026-03-13
**Priority**: CRITICAL
**Auto-fixable**: No
