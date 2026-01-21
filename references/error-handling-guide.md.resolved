# Go Error Handling Guide

Comprehensive guide to idiomatic error handling in Go, based on "Learning Go" by Jon Bodner.

---

## Table of Contents

1. [Basic Error Handling](#basic-error-handling)
2. [Error Wrapping](#error-wrapping)
3. [Sentinel Errors](#sentinel-errors)
4. [Custom Error Types](#custom-error-types)
5. [errors.Is and errors.As](#errorsis-and-errorsas)
6. [Error Design Patterns](#error-design-patterns)
7. [Panic and Recover](#panic-and-recover)
8. [Best Practices](#best-practices)

---

## Basic Error Handling

### The Go Way

Go treats errors as values, not exceptions. Functions return errors as the last return value.

```go
// Standard error handling pattern
func ReadFile(filename string) ([]byte, error) {
    data, err := os.ReadFile(filename)
    if err != nil {
        return nil, err
    }
    return data, nil
}
```

### Always Check Errors

```go
// ❌ NEVER ignore errors
data, _ := ReadFile("config.json")  // DANGEROUS!

// ✅ ALWAYS check
data, err := ReadFile("config.json")
if err != nil {
    return fmt.Errorf("failed to read config: %w", err)
}
```

---

## Error Wrapping

### Use %w for Wrapping (Go 1.13+)

**Why**: Preserves error chain for debugging and allows unwrapping

```go
// ❌ BAD - Loses original error
if err != nil {
    return fmt.Errorf("database query failed: %v", err)
}

// ✅ GOOD - Preserves error chain
if err != nil {
    return fmt.Errorf("database query failed: %w", err)
}
```

### When to Wrap

Wrap errors when:
- Adding context about what operation failed
- Passing errors up the call stack
- You want callers to inspect the underlying error

```go
func (s *Service) GetUser(id int) (*User, error) {
    user, err := s.repo.FindByID(id)
    if err != nil {
        return nil, fmt.Errorf("get user %d: %w", id, err)
    }
    return user, nil
}
```

### Error Stack Trace

```go
// Each layer adds context
// Layer 3 (usecase): "get user 123: repository query failed: sql: no rows"
// Layer 2 (repository): "repository query failed: sql: no rows"
// Layer 1 (database): "sql: no rows"
```

---

## Sentinel Errors

### Definition

Pre-declared package-level error variables for specific conditions.

```go
// Define sentinel errors
var (
    ErrNotFound     = errors.New("resource not found")
    ErrUnauthorized = errors.New("unauthorized access")
    ErrInvalidInput = errors.New("invalid input")
)
```

### Usage Pattern

```go
func (r *Repository) GetByID(id int) (*Entity, error) {
    var entity Entity
    err := r.db.First(&entity, id).Error
    if err != nil {
        if errors.Is(err, gorm.ErrRecordNotFound) {
            return nil, ErrNotFound  // Return sentinel
        }
        return nil, fmt.Errorf("query failed: %w", err)
    }
    return &entity, nil
}

// Caller checks sentinel
entity, err := repo.GetByID(123)
if errors.Is(err, ErrNotFound) {
    // Handle not found case
}
```

### Best Practices for Sentinels

✅ **Do**:
- Export them (capitalize) so callers can check
- Use descriptive names starting with `Err`
- Document when functions might return them
- Use for stable, application-wide error categories

❌ **Don't**:
- Create too many (causes API bloat)
- Use for errors that need additional data
- Use `==` to compare (use `errors.Is` instead)

---

## Custom Error Types

### When to Use

When you need to attach structured data to errors.

```go
// Custom error type
type ValidationError struct {
    Field   string
    Message string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation failed on %s: %s", e.Field, e.Message)
}

// Usage
func ValidateUser(u *User) error {
    if u.Email == "" {
        return &ValidationError{
            Field:   "email",
            Message: "email is required",
        }
    }
    return nil
}
```

### Extracting Custom Errors

```go
// Caller extracts custom error data
err := ValidateUser(user)
var verr *ValidationError
if errors.As(err, &verr) {
    fmt.Printf("Field: %s, Message: %s\n", verr.Field, verr.Message)
}
```

---

## errors.Is and errors.As

### errors.Is - Check for Specific Error

Works with wrapped errors!

```go
err := someFunction()

// ✅ CORRECT - Works with wrapped errors
if errors.Is(err, ErrNotFound) {
    // Handle not found
}

// ❌ WRONG - Doesn't work with wrapped errors
if err == ErrNotFound {
    // Only matches exact error, not wrapped
}
```

### errors.As - Extract Error Type

```go
err := someFunction()

// Extract custom error type
var verr *ValidationError
if errors.As(err, &verr) {
    // verr now contains the ValidationError
    fmt.Println(verr.Field)
}
```

---

## Error Design Patterns

### Pattern 1: Domain Errors

Define errors in domain layer, return in all layers.

```go
// domain/errors.go
var (
    ErrUserNotFound    = errors.New("user not found")
    ErrInvalidPassword = errors.New("invalid password")
)

// repository returns domain errors
func (r *UserRepo) FindByEmail(email string) (*User, error) {
    var user User
    if err := r.db.Where("email = ?", email).First(&user).Error; err != nil {
        if errors.Is(err, gorm.ErrRecordNotFound) {
            return nil, domain.ErrUserNotFound
        }
        return nil, err
    }
    return &user, nil
}

// usecase wraps with context
func (uc *AuthUsecase) Login(email, password string) error {
    user, err := uc.userRepo.FindByEmail(email)
    if err != nil {
        return fmt.Errorf("login failed: %w", err)
    }
    // ...
}
```

### Pattern 2: Error Context

Add contextual information at each layer.

```go
// Layer 3 (handler)
func (h *Handler) GetUser(c echo.Context) error {
    id, _ := strconv.Atoi(c.Param("id"))
    user, err := h.usecase.GetUser(c.Request().Context(), id)
    if err != nil {
        return fmt.Errorf("GET /users/%d: %w", id, err)
    }
    return c.JSON(200, user)
}

// Layer 2 (usecase)
func (uc *Usecase) GetUser(ctx context.Context, id int) (*User, error) {
    user, err := uc.repo.GetByID(ctx, id)
    if err != nil {
        return nil, fmt.Errorf("get user usecase: %w", err)
    }
    return user, nil
}

// Layer 1 (repository)
func (r *Repo) GetByID(ctx context.Context, id int) (*User, error) {
    var user User
    if err := r.db.WithContext(ctx).First(&user, id).Error; err != nil {
        return nil, fmt.Errorf("query user id=%d: %w", id, err)
    }
    return &user, nil
}

// Final error message:
// "GET /users/123: get user usecase: query user id=123: record not found"
```

---

## Panic and Recover

### When to Panic

Use panic only for:
- Initialization errors (can't continue)
- Programming errors (invariants violated)

```go
// ✅ Acceptable panic - initialization
func init() {
    if _, err := loadConfig(); err != nil {
        panic(fmt.Sprintf("failed to load config: %v", err))
    }
}

// ❌ DON'T panic for normal errors
func GetUser(id int) *User {
    user, err := db.FindUser(id)
    if err != nil {
        panic(err)  // WRONG! Return error instead
    }
    return user
}
```

### Recover Pattern

```go
func SafeHandler(w http.ResponseWriter, r *http.Request) {
    defer func() {
        if err := recover(); err != nil {
            log.Printf("Panic recovered: %v", err)
            http.Error(w, "Internal Server Error", 500)
        }
    }()
    
    // Handler logic that might panic
    riskyOperation()
}
```

---

## Best Practices

### 1. Check Errors Immediately

```go
// ✅ GOOD
file, err := os.Open("data.txt")
if err != nil {
    return err
}
defer file.Close()

// ❌ BAD - Delayed check
file, err := os.Open("data.txt")
defer file.Close()  // file might be nil!
if err != nil {
    return err
}
```

### 2. Use errors.Is/As Not ==

```go
// ✅ GOOD - Works with wrapped errors
if errors.Is(err, ErrNotFound) {
    // Handle
}

// ❌ BAD - Only works with exact match
if err == ErrNotFound {
    // Won't match wrapped errors
}
```

### 3. Don't Shadow err

```go
// ❌ BAD - Shadows outer err
err := doFirst()
if err != nil {
    if err := doSecond(); err != nil {  // Shadows!
        return err  // Returns doSecond error, loses doFirst
    }
}

// ✅ GOOD - Use different names
err := doFirst()
if err != nil {
    err2 := doSecond()
    if err2 != nil {
        return fmt.Errorf("doSecond: %w, after doFirst: %v", err2, err)
    }
}
```

### 4. Wrap with Context

```go
// ✅ GOOD - Adds useful context
if err != nil {
    return fmt.Errorf("failed to process order %d: %w", orderID, err)
}

// ❌ BAD - Generic message
if err != nil {
    return fmt.Errorf("error: %w", err)
}
```

### 5. Return Early

```go
// ✅ GOOD - Early return
func Process() error {
    if err := step1(); err != nil {
        return err
    }
    if err := step2(); err != nil {
        return err
    }
    return step3()
}

// ❌ BAD - Deeply nested
func Process() error {
    if err := step1(); err == nil {
        if err := step2(); err == nil {
            return step3()
        } else {
            return err
        }
    }
    return err
}
```

---

## Decision Tree

**Should I wrap this error?**
- Adding context? → Yes, use `%w`
- Just passing up? → Yes, use `%w`
- Converting to domain error? → No, return sentinel/custom error

**Should I use a sentinel error?**
- Callers need to check this specific condition? → Yes
- Error needs additional data? → No, use custom type
- Library/package boundary? → Probably yes

**Should I use a custom error type?**
- Need to attach structured data? → Yes
- Simple error message sufficient? → No, use sentinel or `errors.New`

---

**Based on**: "Learning Go" by Jon Bodner, Chapter 8  
**Last Updated**: 2026-01-21
