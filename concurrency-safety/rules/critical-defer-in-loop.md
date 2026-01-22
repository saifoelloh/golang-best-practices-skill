---
title: Avoid Defer in Loops
impact: CRITICAL
impactDescription: Resource leaks - defer doesn't execute until function returns, not iteration
tags: defer, loops, resource-leak, file-handles, memory-leak
source: Learning Go - Chapter 5 (Functions)
---

## Avoid Defer in Loops

Never use `defer` inside loops. Defer statements accumulate and execute at function return, not at the end of each iteration. This causes resource leaks when processing many items.

**Impact**: CRITICAL - Memory and file descriptor leaks in production

### Detection

How to spot this issue:
- `defer` keyword inside `for`, `for range`, or `while` loops
- Pattern: `for ... { defer ... }`
- Multiple file opens with `defer Close()` in loop
- Growing memory usage during iteration

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Opens 1000 files, defers close at function end
func processFiles(filenames []string) error {
    results := make([]Result, 0, len(filenames))
    
    for _, filename := range filenames {
        file, err := os.Open(filename)
        if err != nil {
            return fmt.Errorf("open %s: %w", filename, err)
        }
        defer file.Close()  // LEAK! Doesn't close until processFiles returns
        
        data, err := io.ReadAll(file)
        if err != nil {
            return fmt.Errorf("read %s: %w", filename, err)
        }
        
        result := process(data)
        results = append(results, result)
    }
    
    return saveResults(results)
}

// With 1000 files:
// - Opens file 1, defers close
// - Opens file 2, defers close
// - ... (all 1000 files stay open!)
// - Returns from function
// - Now all 1000 defers execute
// Result: ulimit exceeded, "too many open files" error
```

**Why this is wrong**:
- All file handles stay open until function returns
- Can exhaust system file descriptors (ulimit)
- Memory not released during iteration
- Can cause crashes in production with large datasets

### Correct (Best Practice)

**Solution 1: Extract to Helper Function**

```go
// ✅ GOOD - Helper function with defer
func processFiles(filenames []string) error {
    results := make([]Result, 0, len(filenames))
    
    for _, filename := range filenames {
        result, err := processFile(filename)
        if err != nil {
            return err
        }
        results = append(results, result)
    }
    
    return saveResults(results)
}

func processFile(filename string) (Result, error) {
    file, err := os.Open(filename)
    if err != nil {
        return Result{}, fmt.Errorf("open %s: %w", filename, err)
    }
    defer file.Close()  // Closes at end of THIS function (each iteration)
    
    data, err := io.ReadAll(file)
    if err != nil {
        return Result{}, fmt.Errorf("read %s: %w", filename, err)
    }
    
    return process(data), nil
}
```

**Solution 2: Anonymous Function**

```go
// ✅ GOOD - Anonymous function creates new scope
func processFiles(filenames []string) error {
    results := make([]Result, 0, len(filenames))
    
    for _, filename := range filenames {
        err := func() error {
            file, err := os.Open(filename)
            if err != nil {
                return fmt.Errorf("open %s: %w", filename, err)
            }
            defer file.Close()  // Closes at end of anonymous function
            
            data, err := io.ReadAll(file)
            if err != nil {
                return fmt.Errorf("read %s: %w", filename, err)
            }
            
            results = append(results, process(data))
            return nil
        }()
        
        if err != nil {
            return err
        }
    }
    
    return saveResults(results)
}
```

**Solution 3: Explicit Close (When Safe)**

```go
// ✅ ACCEPTABLE - Explicit close (only if no early returns)
func processFiles(filenames []string) error {
    results := make([]Result, 0, len(filenames))
    
    for _, filename := range filenames {
        file, err := os.Open(filename)
        if err != nil {
            return fmt.Errorf("open %s: %w", filename, err)
        }
        
        data, err := io.ReadAll(file)
        file.Close()  // Close immediately after use
        if err != nil {
            return fmt.Errorf("read %s: %w", filename, err)
        }
        
        results = append(results, process(data))
    }
    
    return saveResults(results)
}
```

**Why these are better**:
- Resources released immediately after use
- No accumulation of open file handles
- Bounded memory usage
- No risk of hitting system limits

### Real-World Impact

**Scenario**: Processing 10,000 log files

```go
// ❌ BAD - With defer in loop
// File descriptor usage: 10,000 (all open simultaneously)
// Memory: High (all buffers in memory)
// Result: Crashes after ~1,024 files (ulimit)

// ✅ GOOD - With helper function
// File descriptor usage: 1 (one at a time)
// Memory: Low (one buffer at a time)
// Result: Successfully processes all 10,000 files
```

### Additional Context

**What about other resources?**

Same issue applies to:
- Database connections: `defer conn.Close()`
- HTTP response bodies: `defer resp.Body.Close()`
- Mutex locks: `defer mu.Unlock()`
- Channels: `defer close(ch)`

**Example with DB connections**:

```go
// ❌ BAD - Exhausts connection pool
func queryUsers(ids []int) ([]*User, error) {
    users := make([]*User, 0, len(ids))
    
    for _, id := range ids {
        conn, err := db.Acquire()
        if err != nil {
            return nil, err
        }
        defer conn.Release()  // LEAK! Pool exhausted
        
        user, err := conn.QueryUser(id)
        if err != nil {
            return nil, err
        }
        users = append(users, user)
    }
    
    return users, nil
}

// ✅ GOOD - Releases connection each iteration
func queryUsers(ids []int) ([]*User, error) {
    users := make([]*User, 0, len(ids))
    
    for _, id := range ids {
        user, err := queryUser(id)
        if err != nil {
            return nil, err
        }
        users = append(users, user)
    }
    
    return users, nil
}

func queryUser(id int) (*User, error) {
    conn, err := db.Acquire()
    if err != nil {
        return nil, err
    }
    defer conn.Release()
    
    return conn.QueryUser(id)
}
```

### Performance Comparison

```go
// Processing 1000 files
// ❌ defer in loop: Peak memory 500MB, crashes
// ✅ helper function: Peak memory 5MB, completes in 2s
```

### Common Variations

**Don't do this either**:

```go
// ❌ Still bad - goroutine doesn't help
for _, file := range files {
    f, _ := os.Open(file)
    go func() {
        defer f.Close()  // Defers still accumulate!
        process(f)
    }()
}

// ✅ Use helper function or explicit close
```

### Quick Decision Tree

**Is there a `defer` in my loop?**
- Yes → **Extract to helper function**
- Is the resource used only in loop? → **Consider explicit close**
- Need defer for cleanup? → **Use anonymous function**

### References

- "Learning Go" by Jon Bodner - Chapter 5: Functions (Defer section)
- [Go by Example: Defer](https://gobyexample.com/defer)
- [Effective Go: Defer](https://go.dev/doc/effective_go#defer)

---

**Rule Version**: 1.0  
**Last Updated**: 2026-01-21  
**Priority**: CRITICAL  
**Auto-fixable**: Partially (linter can detect, suggest helper function)
