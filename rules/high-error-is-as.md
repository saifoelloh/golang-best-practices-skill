---
title: Use errors.Is/As Instead of ==
impact: HIGH
impactDescription: Direct comparison fails with wrapped errors, breaking error handling logic
tags: error-handling, errors.Is, errors.As, wrapping, comparison
source: Learning Go - Chapter 8 (Errors)
---

## Use errors.Is/As Instead of ==

Never use `==` or `!=` to compare errors. Use `errors.Is()` for sentinel errors and `errors.As()` for custom error types. Direct comparison fails when errors are wrapped.

**Impact**: HIGH - Error handling logic breaks with wrapped errors

### Detection

How to spot this issue:
- Direct error comparison: `err == ErrNotFound`
- Negation: `err != nil && err != ErrTimeout`
- Pattern: `if err == SomeError` instead of `errors.Is(err, SomeError)`

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Direct comparison fails with wrapped errors
var ErrNotFound = errors.New("not found")

func GetUser(id int) (*User, error) {
    user, err := db.Query(id)
    if err != nil {
        return nil, fmt.Errorf("query failed: %w", err)  // Wrapped!
    }
    
    if user == nil {
        return nil, ErrNotFound  // Returns sentinel
    }
    
    return user, nil
}

// Caller - BROKEN!
user, err := GetUser(123)
if err == ErrNotFound {  // NEVER matches wrapped errors!
    return "User not found"
}
```

**Why this is wrong**:
- `==` only matches exact error value
- Wrapped errors have different memory addresses
- Error handling logic never executes
- Silent failures in production

### Correct (Best Practice)

```go
// ✅ GOOD - errors.Is works with wrapped errors
var ErrNotFound = errors.New("not found")

func GetUser(id int) (*User, error) {
    user, err := db.Query(id)
    if err != nil {
        return nil, fmt.Errorf("query failed: %w", err)  // Wrapped
    }
    
    if user == nil {
        return nil, fmt.Errorf("user %d: %w", id, ErrNotFound)  // Wrapped sentinel
    }
    
    return user, nil
}

// Caller - Works!
user, err := GetUser(123)
if errors.Is(err, ErrNotFound) {  // Matches even when wrapped!
    return "User not found"
}
```

**Why this is better**:
- `errors.Is()` unwraps error chain
- Finds sentinel anywhere in chain
- Error handling works correctly
- Robust against wrapping

### Using errors.As for Custom Types

```go
// Custom error type
type ValidationError struct {
    Field   string
    Message string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("%s: %s", e.Field, e.Message)
}

// ❌ BAD - Type assertion on wrapped error
func ProcessInput(input string) error {
    err := validate(input)
    if err != nil {
        return fmt.Errorf("validation failed: %w", err)
    }
    return nil
}

// Caller - BROKEN!
err := ProcessInput("bad")
if verr, ok := err.(*ValidationError); ok {  // Never matches wrapped!
    fmt.Printf("Field: %s\n", verr.Field)
}
```

```go
// ✅ GOOD - errors.As extracts from chain
func ProcessInput(input string) error {
    err := validate(input)
    if err != nil {
        return fmt.Errorf("validation failed: %w", err)
    }
    return nil
}

// Caller - Works!
err := ProcessInput("bad")
var verr *ValidationError
if errors.As(err, &verr) {  // Finds ValidationError in chain!
    fmt.Printf("Field: %s\n", verr.Field)
}
```

### Real-World Example

```go
// Domain errors
var (
    ErrNotFound     = errors.New("resource not found")
    ErrUnauthorized = errors.New("unauthorized")
    ErrConflict     = errors.New("resource conflict")
)

// ❌ BAD - Repository with broken error checking
func (r *UserRepo) GetByEmail(email string) (*User, error) {
    var user User
    err := r.db.Where("email = ?", email).First(&user).Error
    
    if err == gorm.ErrRecordNotFound {  // GORM error
        return nil, ErrNotFound  // Return domain error
    }
    
    if err != nil {
        return nil, fmt.Errorf("query failed: %w", err)
    }
    
    return &user, nil
}

// Service - BROKEN comparison
func (s *Service) Login(email, password string) error {
    user, err := s.repo.GetByEmail(email)
    if err == ErrNotFound {  // Works (not wrapped)
        return errors.New("invalid credentials")
    }
    // ... But if repo wraps the error, this breaks!
}
```

```go
// ✅ GOOD - Robust error checking
func (r *UserRepo) GetByEmail(email string) (*User, error) {
    var user User
    err := r.db.Where("email = ?", email).First(&user).Error
    
    if errors.Is(err, gorm.ErrRecordNotFound) {  // Use errors.Is
        return nil, fmt.Errorf("email %s: %w", email, ErrNotFound)
    }
    
    if err != nil {
        return nil, fmt.Errorf("query failed: %w", err)
    }
    
    return &user, nil
}

// Service - Works even with wrapped errors
func (s *Service) Login(email, password string) error {
    user, err := s.repo.GetByEmail(email)
    if errors.Is(err, ErrNotFound) {  // Robust!
        return errors.New("invalid credentials")
    }
    // ...
}
```

### Multiple Error Checks

```go
// ❌ BAD - Multiple direct comparisons
if err != nil {
    if err == ErrNotFound || err == ErrDeleted {
        return http.StatusNotFound
    }
    if err == ErrUnauthorized {
        return http.StatusUnauthorized
    }
}

// ✅ GOOD - errors.Is for each
if err != nil {
    if errors.Is(err, ErrNotFound) || errors.Is(err, ErrDeleted) {
        return http.StatusNotFound
    }
    if errors.Is(err, ErrUnauthorized) {
        return http.StatusUnauthorized
    }
}
```

### Network Errors Example

```go
// ✅ Checking for specific network errors
resp, err := http.Get(url)
if err != nil {
    var netErr *net.OpError
    if errors.As(err, &netErr) {
        if netErr.Timeout() {
            return errors.New("request timeout")
        }
    }
    
    var dnsErr *net.DNSError
    if errors.As(err, &dnsErr) {
        return fmt.Errorf("DNS resolution failed: %w", err)
    }
    
    return err
}
```

### Additional Context

**How errors.Is works**:

```go
// Simplified implementation
func Is(err, target error) bool {
    for {
        if err == target {
            return true
        }
        
        // Check if err implements Unwrap
        unwrapper, ok := err.(interface{ Unwrap() error })
        if !ok {
            return false
        }
        
        err = unwrapper.Unwrap()
        if err == nil {
            return false
        }
    }
}
```

**How errors.As works**:

```go
// Simplified implementation
func As(err error, target interface{}) bool {
    for {
        if reflect.TypeOf(err) == reflect.TypeOf(target).Elem() {
            reflect.ValueOf(target).Elem().Set(reflect.ValueOf(err))
            return true
        }
        
        unwrapper, ok := err.(interface{ Unwrap() error })
        if !ok {
            return false
        }
        
        err = unwrapper.Unwrap()
        if err == nil {
            return false
        }
    }
}
```

### Common Patterns

**HTTP error handling**:

```go
// ✅ Convert domain errors to HTTP status
func handleError(err error) int {
    if errors.Is(err, ErrNotFound) {
        return http.StatusNotFound
    }
    if errors.Is(err, ErrUnauthorized) {
        return http.StatusUnauthorized
    }
    if errors.Is(err, ErrConflict) {
        return http.StatusConflict
    }
    
    var validationErr *ValidationError
    if errors.As(err, &validationErr) {
        return http.StatusBadRequest
    }
    
    return http.StatusInternalServerError
}
```

### Quick Decision Tree

**Comparing to sentinel error?**
- Yes → **Use `errors.Is(err, ErrSentinel)`**

**Extracting custom error type?**
- Yes → **Use `errors.As(err, &customErr)`**

**Checking if error is nil?**
- Use `err != nil` → **This is fine, not comparing values**

**Checklist**:
- [ ] No `err == ErrSomething` comparisons
- [ ] Use `errors.Is()` for sentinel checks
- [ ] Use `errors.As()` for type extraction
- [ ] All error checks work with wrapped errors

### References

- [Go Blog: Working with Errors in Go 1.13](https://go.dev/blog/go1.13-errors)
- [pkg.go.dev/errors](https://pkg.go.dev/errors)
- "Learning Go" by Jon Bodner - Chapter 8: Errors

---

**Rule Version**: 1.0  
**Last Updated**: 2026-01-21  
**Priority**: HIGH  
**Auto-fixable**: Yes (linter can detect and suggest fix)
