---
title: Don't Shadow err Variable
impact: CRITICAL
impactDescription: Silently loses errors, making bugs invisible and impossible to debug
tags: error-handling, shadowing, scope, variables, debugging
source: Learning Go - Chapter 4 (Blocks, Shadows)
---

## Don't Shadow err Variable

Never shadow the `err` variable in nested scopes using `:=`. Shadowing causes the outer error to be hidden, potentially losing critical error information.

**Impact**: CRITICAL - Silent error loss, making failures invisible

### Detection

How to spot this issue:
- Nested `if err := ...` or `err := ...` statements
- Inner scope declares new `err` variable
- Outer `err` is never checked or returned
- Pattern: `err :=` inside another `if err != nil` block

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Shadows err variable
func ProcessOrder(orderID int) error {
    order, err := getOrder(orderID)
    if err != nil {
        // Try to log the error
        if err := logError(err); err != nil {  // SHADOWS outer err!
            return err  // Returns log error, loses getOrder error!
        }
        // Outer err is shadowed, never returned!
    }
    
    return processPayment(order)
}

// Scenario:
// 1. getOrder fails with "database connection lost"
// 2. logError fails with "log file full"
// 3. Function returns "log file full"
// 4. Original "database connection lost" error is LOST!
```

**Why this is wrong**:
- Inner `:=` creates new `err` variable in inner scope
- Outer `err` is hidden/shadowed
- Original error is never returned or handled
- Debugging becomes impossible (wrong error reported)

### Correct (Best Practice)

**Solution 1: Use Different Variable Names**

```go
// ✅ GOOD - Use different variable name
func ProcessOrder(orderID int) error {
    order, err := getOrder(orderID)
    if err != nil {
        // Use different variable name
        logErr := logError(err)
        if logErr != nil {
            log.Printf("Failed to log error: %v", logErr)
        }
        return fmt.Errorf("failed to get order: %w", err)  // Returns original error!
    }
    
    return processPayment(order)
}
```

**Solution 2: Use Assignment (=) Instead of Declaration (:=)**

```go
// ✅ GOOD - Reuse err variable with =
func ProcessOrder(orderID int) error {
    order, err := getOrder(orderID)
    if err != nil {
        return fmt.Errorf("failed to get order: %w", err)
    }
    
    // Reuses existing err variable
    err = validateOrder(order)
    if err != nil {
        return fmt.Errorf("validation failed: %w", err)
    }
    
    return processPayment(order)
}
```

**Solution 3: Separate Error Handling**

```go
// ✅ GOOD - Handle errors sequentially
func ProcessOrder(orderID int) error {
    order, err := getOrder(orderID)
    if err != nil {
        logError(err)  // Ignore log errors
        return fmt.Errorf("failed to get order: %w", err)
    }
    
    if err := validateOrder(order); err != nil {
        return fmt.Errorf("validation failed: %w", err)
    }
    
    return processPayment(order)
}
```

**Why these are better**:
- No variable shadowing
- Original errors properly returned
- Clear error flow
- Easy to debug

### Real-World Example

```go
// ❌ BAD - Production bug from shadowing
func (s *Service) CreateUser(ctx context.Context, req *CreateUserRequest) error {
    // Validate input
    user, err := s.validator.Validate(req)
    if err != nil {
        // Shadow bug: tries to save anyway!
        if err := s.repo.SaveAuditLog(ctx, "validation_failed"); err != nil {
            return err  // Returns audit error, validation error lost!
        }
        // BUG: Never returns validation error!
        // Continues to save invalid user!
    }
    
    // This executes even if validation failed!
    return s.repo.SaveUser(ctx, user)
}

// Result:
// - Invalid users saved to database
// - Production data corruption
// - Error logs say "failed to save audit" (misleading!)
// - Root cause (validation failure) is invisible
```

```go
// ✅ GOOD - Proper error handling
func (s *Service) CreateUser(ctx context.Context, req *CreateUserRequest) error {
    user, err := s.validator.Validate(req)
    if err != nil {
        // Use different variable
        auditErr := s.repo.SaveAuditLog(ctx, "validation_failed")
        if auditErr != nil {
            log.Printf("Failed to save audit: %v", auditErr)
        }
        return fmt.Errorf("validation failed: %w", err)  // Returns validation error
    }
    
    return s.repo.SaveUser(ctx, user)
}
```

### Multiple Shadow Levels

```go
// ❌ VERY BAD - Multiple levels of shadowing
func ComplexOperation() error {
    data, err := fetchData()
    if err != nil {
        if err := tryCache(); err != nil {  // Shadow level 1
            if err := logError(err); err != nil {  // Shadow level 2
                return err  // Which err? The log error!
            }
        }
        // Both fetchData and tryCache errors lost!
    }
    return processData(data)
}

// ✅ GOOD - Clear error handling
func ComplexOperation() error {
    data, err := fetchData()
    if err != nil {
        cacheErr := tryCache()
        if cacheErr != nil {
            log.Printf("Cache failed: %v", cacheErr)
        }
        
        logErr := logError(err)
        if logErr != nil {
            log.Printf("Logging failed: %v", logErr)
        }
        
        return fmt.Errorf("failed to fetch data: %w", err)
    }
    
    return processData(data)
}
```

### Loop Shadowing

```go
// ❌ BAD - Shadowing in loop
func ProcessFiles(files []string) error {
    var err error
    
    for _, file := range files {
        if err := processFile(file); err != nil {  // Shadows outer err!
            log.Printf("Failed to process %s", file)
            // Outer err never set, always nil!
        }
    }
    
    return err  // Always returns nil!
}

// ✅ GOOD - Proper error accumulation
func ProcessFiles(files []string) error {
    var firstErr error
    
    for _, file := range files {
        err := processFile(file)
        if err != nil {
            log.Printf("Failed to process %s: %v", file, err)
            if firstErr == nil {
                firstErr = err  // Track first error
            }
        }
    }
    
    if firstErr != nil {
        return fmt.Errorf("file processing failed: %w", firstErr)
    }
    return nil
}
```

### Linter Detection

Most linters can detect shadowing:

```bash
# Using golangci-lint
golangci-lint run --enable=govet

# Using go vet
go vet ./...

# Output:
# main.go:15:7: declaration of "err" shadows declaration at line 10
```

### Additional Context

**Why `:=` causes shadowing**:

```go
// := creates NEW variable in current scope
if err := doSomething(); err != nil {
    // 'err' here is different from outer 'err'
}

// = assigns to EXISTING variable
err = doSomething()
if err != nil {
    // 'err' here is the same as outer 'err'
}
```

**Safe shadowing (different variable)**:

```go
// ✅ OK - Shadowing 'i' is fine (not err)
for i := 0; i < 10; i++ {
    if i > 5 {
        i := i * 2  // Shadows loop variable - OK
        fmt.Println(i)
    }
}
```

### Quick Decision Tree

**Am I using `:=` with `err` variable?**
- Yes → **Is there an outer `err` in scope?**
  - Yes → **DON'T shadow! Use different name**
  - No → **OK to use `:=`**
- Using `=` → **OK, reuses existing err**

**Checklist**:
- [ ] No nested `err :=` declarations
- [ ] Different variable names for different errors
- [ ] All errors properly returned or logged
- [ ] Run `go vet` to catch shadows

### References

- "Learning Go" by Jon Bodner - Chapter 4: Blocks, Shadows
- [Go Vet: Shadow](https://pkg.go.dev/golang.org/x/tools/go/analysis/passes/shadow)
- [Effective Go: Redeclaration](https://go.dev/doc/effective_go#redeclaration)

---

**Rule Version**: 1.0  
**Last Updated**: 2026-01-21  
**Priority**: CRITICAL  
**Auto-fixable**: Yes (linter can detect and suggest fix)
