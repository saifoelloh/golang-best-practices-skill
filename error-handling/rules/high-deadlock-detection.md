---
title: Handle deadlock_detected (40P01) Separately from Other Errors
impact: HIGH
impactDescription: Deadlocks are retryable — treating them as fatal errors wastes valid transactions
tags: postgresql, deadlock, 40P01, retry, transaction, concurrency
source: PostgreSQL Docs - Deadlocks
---

## Handle deadlock_detected (40P01) Separately from Other Errors

PostgreSQL detects deadlocks and automatically aborts one of the conflicting transactions with `ERROR 40P01: deadlock detected`. Like `40001` (serialization failure), this is a retryable condition — the aborted transaction should be retried after a short delay.

**Impact**: HIGH - Legitimate transactions silently fail as generic errors instead of being retried

### Detection

- `40P01` error not handled separately in error mapping
- Deadlock errors logged as fatal without retry attempt

### Correct (Best Practice)

```go
// ✅ GOOD - deadlock and serialization failure both handled as retryable
func isRetryable(err error) bool {
    var pgErr *pgconn.PgError
    if errors.As(err, &pgErr) {
        return pgErr.Code == "40001" || pgErr.Code == "40P01"
    }
    return false
}

func WithRetry(db *gorm.DB, maxRetries int, fn func(*gorm.DB) error) error {
    for attempt := 1; attempt <= maxRetries; attempt++ {
        err := db.Transaction(fn)
        if err == nil {
            return nil
        }
        if isRetryable(err) && attempt < maxRetries {
            time.Sleep(time.Duration(attempt) * 50 * time.Millisecond)
            continue
        }
        return err
    }
    return ErrMaxRetriesExceeded
}
```

### Prevention

Deadlocks often happen when multiple transactions lock rows in different orders. Fix by ensuring consistent lock ordering:

```go
// ✅ Always lock rows in ascending ID order to prevent deadlocks
db.Transaction(func(tx *gorm.DB) error {
    // Lock lower ID first, then higher ID — always consistent
    ids := []uint{id1, id2}
    sort.Slice(ids, func(i, j int) bool { return ids[i] < ids[j] })
    for _, id := range ids {
        tx.Set("gorm:query_option", "FOR UPDATE").First(&record, id)
    }
    return nil
})
```

### References

- [PostgreSQL Docs: Deadlocks](https://www.postgresql.org/docs/16/explicit-locking.html#LOCKING-DEADLOCKS)

---

**Rule Version**: 1.0
**Last Updated**: 2026-03-13
**Priority**: HIGH
**Auto-fixable**: No
