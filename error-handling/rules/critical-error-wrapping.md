---
title: Use %w for Error Wrapping
impact: CRITICAL
impactDescription: Without %w, error chains are lost, breaking errors.Is/As and stack traces
tags: error-handling, wrapping, debugging, fmt.Errorf
source: Learning Go - Chapter 8 (Errors)
---

## Use %w for Error Wrapping

Always use `%w` when wrapping errors with `fmt.Errorf`, not `%v`. This preserves the error chain for debugging and allows callers to inspect the underlying error using `errors.Is` and `errors.As`.

**Impact**: CRITICAL - Without proper wrapping, stack traces are lost and error inspection fails

### Detection

How to spot this issue:
- `fmt.Errorf` calls using `%v` instead of `%w`
- Pattern: `fmt.Errorf("message: %v", err)`
- Loss of ability to use `errors.Is()` or `errors.As()`

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Uses %v, loses error chain
func (s *Service) ProcessOrder(orderID int) error {
    order, err := s.repo.GetOrder(orderID)
    if err != nil {
        return fmt.Errorf("failed to get order: %v", err)
        // Error chain is broken! Can't use errors.Is anymore
    }
    
    if err := s.validateOrder(order); err != nil {
        return fmt.Errorf("validation failed: %v", err)
        // Original error type is lost
    }
    
    return nil
}

// Caller can't check underlying error
err := service.ProcessOrder(123)
if errors.Is(err, sql.ErrNoRows) {
    // This will NEVER match because error chain was broken!
}
```

**Why this is wrong**:
- `%v` converts error to string, losing the original error object
- `errors.Is()` and `errors.As()` can't inspect the chain
- Debugging becomes extremely difficult without full stack trace
- Production issues are harder to diagnose

### Correct (Best Practice)

```go
// ✅ GOOD - Uses %w, preserves error chain
func (s *Service) ProcessOrder(orderID int) error {
    order, err := s.repo.GetOrder(orderID)
    if err != nil {
        return fmt.Errorf("failed to get order: %w", err)
        // Error chain preserved!
    }
    
    if err := s.validateOrder(order); err != nil {
        return fmt.Errorf("validation failed: %w", err)
        // Can still unwrap to original error
    }
    
    return nil
}

// Caller can check underlying error
err := service.ProcessOrder(123)
if errors.Is(err, sql.ErrNoRows) {
    // This works! errors.Is unwraps the chain
    return nil, status.Error(codes.NotFound, "order not found")
}
```

**Why this is better**:
- Preserves full error chain for debugging
- `errors.Is()` can match errors anywhere in the chain
- `errors.As()` can extract custom error types
- Full context available in logs: "failed to get order: query failed: sql: no rows"

### Error Chain Example

```go
// With %w, you get full chain:
// Layer 3: "failed to get order: query failed: sql: no rows"
// Layer 2: "query failed: sql: no rows"
// Layer 1: "sql: no rows"

// With %v, you only get:
// Layer 3: "failed to get order: query failed: sql: no rows"
// (No way to programmatically check what went wrong)
```

### Additional Context

**When to use %w**:
- Always, when wrapping errors to pass up the stack
- When you want callers to be able to inspect the error
- When adding context to an error

**When NOT to use %w**:
- When you want to hide implementation details from callers
- When converting to a different error domain (use sentinel errors instead)

**Example of intentional hiding**:
```go
// Internal implementation detail - don't expose
if err != nil {
    // Don't wrap - return domain error instead
    return ErrOrderNotFound
}
```

### Impact on Production

**Without %w (Bad)**:
```
Error: failed to process payment
(No idea what the root cause was!)
```

**With %w (Good)**:
```
Error: failed to process payment: stripe charge failed: card declined: insufficient funds
(Clear path to root cause!)
```

### Integration with errors.Is and errors.As

```go
// Define sentinel errors
var (
    ErrNotFound     = errors.New("not found")
    ErrUnauthorized = errors.New("unauthorized")
)

// Repository returns wrapped errors
func (r *Repo) GetByID(id int) (*Entity, error) {
    var entity Entity
    if err := r.db.First(&entity, id).Error; err != nil {
        if errors.Is(err, gorm.ErrRecordNotFound) {
            return nil, fmt.Errorf("entity %d: %w", id, ErrNotFound)
        }
        return nil, fmt.Errorf("query failed: %w", err)
    }
    return &entity, nil
}

// Service adds more context
func (s *Service) GetEntity(id int) (*Entity, error) {
    entity, err := s.repo.GetByID(id)
    if err != nil {
        return nil, fmt.Errorf("get entity: %w", err)
    }
    return entity, nil
}

// Handler can check the sentinel
entity, err := service.GetEntity(123)
if errors.Is(err, ErrNotFound) {
    // Works! Even though wrapped multiple times
    return http.StatusNotFound
}
```

### Quick Decision Tree

**Should I use %w?**
- Adding context to an error? → **Yes**
- Passing error up the stack? → **Yes**
- Want callers to inspect error? → **Yes**
- Hiding implementation details? → **No** (return sentinel error)
- Converting error domains? → **No** (return domain error)

### References

- [Go Blog: Working with Errors in Go 1.13](https://go.dev/blog/go1.13-errors)
- [pkg.go.dev/errors](https://pkg.go.dev/errors)
- "Learning Go" by Jon Bodner - Chapter 8: Errors
- [Effective Go: Errors](https://go.dev/doc/effective_go#errors)

---

**Rule Version**: 1.0  
**Last Updated**: 2026-01-21  
**Priority**: CRITICAL  
**Auto-fixable**: Yes (linter can detect and suggest fix)
