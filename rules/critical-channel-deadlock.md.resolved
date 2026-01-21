---
title: Ensure Paired Send/Receive Channel Operations
impact: CRITICAL
impactDescription: Deadlocks freeze entire program, causing production outages
tags: channel, deadlock, concurrency, blocking, goroutine
source: Concurrency in Go - Chapter 3 (Channels)
---

## Ensure Paired Send/Receive Channel Operations

Every channel send must have a corresponding receive operation, and vice versa. Unpaired operations cause deadlocks that freeze your entire program.

**Impact**: CRITICAL - Deadlocks halt program execution, causing production outages

### Detection

How to spot this issue:
- Unbuffered channel send without receiver
- Channel receive without sender
- Blocking operations in main goroutine
- Runtime error: `fatal error: all goroutines are asleep - deadlock!`

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Send without receiver (deadlock)
func processData() {
    ch := make(chan int)
    ch <- 42  // Blocks forever! No receiver
    
    fmt.Println("Never reached")
}
// Runtime error: fatal error: all goroutines are asleep - deadlock!

// ❌ BAD - Receive without sender (deadlock)
func getData() int {
    ch := make(chan int)
    return <-ch  // Blocks forever! No sender
}

// ❌ BAD - Both in same goroutine (deadlock)
func badPattern() {
    ch := make(chan int)
    ch <- 42       // Blocks waiting for receiver
    val := <-ch    // Never reached!
    fmt.Println(val)
}
```

**Why this is wrong**:
- Unbuffered channels require sender AND receiver ready simultaneously
- Without pair, operation blocks forever
- All goroutines blocked = deadlock panic
- Production service becomes unresponsive

### Correct (Best Practice)

**Solution 1: Send in Goroutine**

```go
// ✅ GOOD - Send in separate goroutine
func processData() {
    ch := make(chan int)
    
    go func() {
        ch <- 42  // Send in goroutine
    }()
    
    val := <-ch  // Receive in main goroutine
    fmt.Println(val)
}
```

**Solution 2: Use Buffered Channel**

```go
// ✅ GOOD - Buffered channel allows send without immediate receiver
func processData() {
    ch := make(chan int, 1)  // Buffer size 1
    
    ch <- 42  // Doesn't block (buffer has space)
    val := <-ch  // Receive later
    fmt.Println(val)
}
```

**Solution 3: Select with Default/Timeout**

```go
// ✅ GOOD - Non-blocking send with select
func trySend(ch chan<- int, value int) bool {
    select {
    case ch <- value:
        return true  // Sent successfully
    default:
        return false  // Would block, skip
    }
}

// ✅ GOOD - Receive with timeout
func receiveWithTimeout(ch <-chan int, timeout time.Duration) (int, error) {
    select {
    case val := <-ch:
        return val, nil
    case <-time.After(timeout):
        return 0, errors.New("timeout")
    }
}
```

**Why these are better**:
- Send and receive happen concurrently
- No blocking operations
- Graceful handling of channel operations
- Production-safe patterns

### Real-World Deadlock Examples

**Example 1: Request Handler Deadlock**

```go
// ❌ BAD - HTTP handler deadlocks
func (h *Handler) ProcessRequest(c echo.Context) error {
    resultCh := make(chan Result)
    
    // Process in background
    go func() {
        result := h.service.Process()
        resultCh <- result  // Might block if handler returns early!
    }()
    
    // Handler might return before receiving
    if err := validateRequest(c); err != nil {
        return err  // Returns WITHOUT receiving! Goroutine leaks & may deadlock
    }
    
    result := <-resultCh
    return c.JSON(200, result)
}
```

```go
// ✅ GOOD - Always receive or use buffered channel
func (h *Handler) ProcessRequest(c echo.Context) error {
    if err := validateRequest(c); err != nil {
        return err  // Early return is safe
    }
    
    resultCh := make(chan Result, 1)  // Buffered!
    
    go func() {
        result := h.service.Process()
        resultCh <- result  // Won't block even if handler returns
    }()
    
    select {
    case result := <-resultCh:
        return c.JSON(200, result)
    case <-c.Request().Context().Done():
        return c.NoContent(499)  // Client disconnected
    }
}
```

**Example 2: Pipeline Deadlock**

```go
// ❌ BAD - Pipeline deadlock
func pipeline() {
    ch1 := make(chan int)
    ch2 := make(chan int)
    
    // Stage 1
    go func() {
        for i := 0; i < 5; i++ {
            ch1 <- i
        }
        // Forgot to close! ch2 receiver waits forever
    }()
    
    // Stage 2
    go func() {
        for val := range ch1 {  // Blocks forever after 5 items
            ch2 <- val * 2
        }
    }()
    
    // Receiver
    for val := range ch2 {
        fmt.Println(val)
    }
    // Deadlock! Stage 2 never closes ch2
}
```

```go
// ✅ GOOD - Close channels properly
func pipeline() {
    ch1 := make(chan int)
    ch2 := make(chan int)
    
    // Stage 1
    go func() {
        defer close(ch1)  // Close when done!
        for i := 0; i < 5; i++ {
            ch1 <- i
        }
    }()
    
    // Stage 2
    go func() {
        defer close(ch2)  // Close output channel!
        for val := range ch1 {
            ch2 <- val * 2
        }
    }()
    
    // Receiver
    for val := range ch2 {
        fmt.Println(val)
    }
    // Works! All channels closed properly
}
```

### Unbuffered vs Buffered Channels

```go
// Unbuffered - requires synchronization
ch := make(chan int)
ch <- 1  // Blocks until receiver ready

// Buffered - asynchronous up to capacity
ch := make(chan int, 10)
ch <- 1  // Doesn't block (buffer has space)
ch <- 2  // Doesn't block
// ... can send 10 items before blocking
```

**When to use each**:
- **Unbuffered**: Strict synchronization needed
- **Buffered**: Decoupling sender/receiver (size = workload capacity)

### Common Deadlock Patterns

**Pattern 1: Send then Receive (same goroutine)**

```go
// ❌ Deadlock
ch := make(chan int)
ch <- 42  // Blocks
val := <-ch  // Never reached

// ✅ Fix: Use goroutine or buffer
ch := make(chan int, 1)
ch <- 42
val := <-ch
```

**Pattern 2: Circular Wait**

```go
// ❌ Deadlock
ch1 := make(chan int)
ch2 := make(chan int)

go func() {
    ch1 <- 1
    <-ch2  // Waits for ch2
}()

go func() {
    ch2 <- 2
    <-ch1  // Waits for ch1
}()
// Both goroutines blocked!

// ✅ Fix: Reorder operations
go func() {
    ch1 <- 1
}()

go func() {
    <-ch1  // Receive first
    ch2 <- 2
}()

<-ch2
```

**Pattern 3: Missing Close**

```go
// ❌ Deadlock
ch := make(chan int)

go func() {
    for i := 0; i < 5; i++ {
        ch <- i
    }
    // Missing close(ch)!
}()

for val := range ch {  // Waits forever after 5
    fmt.Println(val)
}

// ✅ Fix: Close channel
go func() {
    for i := 0; i < 5; i++ {
        ch <- i
    }
    close(ch)  // Signal done!
}()
```

### Debugging Deadlocks

```go
// Use timeouts to detect potential deadlocks
select {
case val := <-ch:
    fmt.Println(val)
case <-time.After(5 * time.Second):
    log.Fatal("Potential deadlock detected!")
}

// Or use context with timeout
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

select {
case val := <-ch:
    fmt.Println(val)
case <-ctx.Done():
    return ctx.Err()  // Timeout or cancellation
}
```

### Additional Context

**Deadlock detection**:
- Go runtime detects when ALL goroutines are blocked
- Panics with "all goroutines are asleep - deadlock!"
- Only detects global deadlock, not subset deadlocks

**Buffer size guidelines**:
```go
// Size 1: Signal/notification
ch := make(chan bool, 1)

// Size = num workers: Work queue
ch := make(chan Task, 10)

// Size = batch size: Batch processing
ch := make(chan Record, 1000)
```

### Quick Decision Tree

**Am I using unbuffered channel?**
- Yes → **Send and receive in different goroutines**
- Using same goroutine? → **Use buffered channel**

**Is channel operation blocking?**
- Yes → **Add select with timeout/default**
- In main goroutine? → **Use goroutine for send/receive**

**Checklist**:
- [ ] Every send has paired receive (or buffer)
- [ ] Every receive has paired send (or will close)
- [ ] Channels closed when no more sends
- [ ] Use select with timeout for safety

### References

- "Concurrency in Go" by Katherine Cox-Buday - Chapter 3: Channels
- [Go by Example: Channels](https://gobyexample.com/channels)
- [Effective Go: Channels](https://go.dev/doc/effective_go#channels)

---

**Rule Version**: 1.0  
**Last Updated**: 2026-01-21  
**Priority**: CRITICAL  
**Auto-fixable**: No (requires design decision)
