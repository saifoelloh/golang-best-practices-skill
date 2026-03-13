---
title: Wrap Database Errors with Operation Context Before Returning
impact: HIGH
impactDescription: Unwrapped DB errors lose call site info — impossible to debug which query failed
tags: gorm, error-wrapping, fmt-errorf, context, debugging, observability
source: Go Blog - Working with Errors
---

## Wrap Database Errors with Operation Context Before Returning

Returning a raw GORM or PostgreSQL error without context makes debugging impossible — the caller sees `"record not found"` or `"pq: duplicate key"` with no indication of which repository method or what parameters caused it.

**Impact**: HIGH - Undebuggable errors in production; no traceability from error to source

### Detection

- Repository methods returning `err` directly without `fmt.Errorf("MethodName: %w", err)`
- Raw `return result.Error` without wrapping

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - no context, impossible to trace
func (r *OrderRepository) FindByID(ctx context.Context, id uint) (*Order, error) {
    var order Order
    if err := r.db.WithContext(ctx).First(&order, id).Error; err != nil {
        return nil, err  // Which method? Which id? Unknown.
    }
    return &order, nil
}
```

### Correct (Best Practice)

```go
// ✅ GOOD - operation + parameter in error message
func (r *OrderRepository) FindByID(ctx context.Context, id uint) (*Order, error) {
    var order Order
    if err := r.db.WithContext(ctx).First(&order, id).Error; err != nil {
        if errors.Is(err, gorm.ErrRecordNotFound) {
            return nil, ErrOrderNotFound
        }
        return nil, fmt.Errorf("OrderRepository.FindByID(id=%d): %w", id, err)
    }
    return &order, nil
}

// ✅ GOOD - structured logging for observability
func (r *OrderRepository) FindByID(ctx context.Context, id uint) (*Order, error) {
    var order Order
    if err := r.db.WithContext(ctx).First(&order, id).Error; err != nil {
        if errors.Is(err, gorm.ErrRecordNotFound) {
            return nil, ErrOrderNotFound
        }
        logger.Error().Err(err).Uint("order_id", id).Msg("OrderRepository.FindByID failed")
        return nil, fmt.Errorf("OrderRepository.FindByID(id=%d): %w", id, err)
    }
    return &order, nil
}
```

### References

- [Go Blog: Working with Errors in Go 1.13](https://go.dev/blog/go1.13-errors)

---

**Rule Version**: 1.0
**Last Updated**: 2026-03-13
**Priority**: HIGH
**Auto-fixable**: Partially
