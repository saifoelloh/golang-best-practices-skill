---
title: Sender Closes Channel, Not Receiver
impact: CRITICAL
impactDescription: Runtime panic from closing wrong side or closing multiple times
tags: channel, close, panic, concurrency, sender-receiver
source: Concurrency in Go - Chapter 3 (Channels)
---

## Sender Closes Channel, Not Receiver

Only the sender should close a channel. Closing from the receiver side or closing multiple times causes runtime panics.

**Impact**: CRITICAL - Runtime panic crashes your program

### Detection

How to spot this issue:
- `close()` called in receiver goroutine
- `close()` called on receive-only channel `<-chan`
- Multiple `close()` calls on same channel
- Pattern: `close(ch)` where `ch` is not being sent to

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Receiver closing channel
func consumer(ch <-chan int) {
    for val := range ch {
        process(val)
    }
    close(ch)  // PANIC! Can't close receive-only channel
}

// ❌ BAD - Multiple closes
func producer(ch chan int) {
    ch <- 42
    close(ch)
    
    // Later in code...
    close(ch)  // PANIC! Channel already closed
}

// ❌ BAD - Closing in wrong goroutine
func badPattern() {
    ch := make(chan int)
    
    // Producer
    go func() {
        ch <- 1
        ch <- 2
        // Forgot to close!
    }()
    
    // Consumer tries to close
    for val := range ch {
        fmt.Println(val)
    }
    close(ch)  // WRONG! Consumer shouldn't close
}
```

**Why this is wrong**:
- Closing receive-only channel: compile error or panic
- Multiple closes: runtime panic "close of closed channel"
- Receiver closing: sender might send to closed channel (panic)
- Violates channel ownership principle

### Correct (Best Practice)

**Solution 1: Sender Closes with defer**

```go
// ✅ GOOD - Sender closes channel
func producer(ch chan<- int) {
    defer close(ch)  // Always close when sender done
    
    for i := 0; i < 10; i++ {
        ch <- i
    }
}

func consumer(ch <-chan int) {
    for val := range ch {  // Exits when channel closes
        fmt.Println(val)
    }
}

// Usage
ch := make(chan int)
go producer(ch)
consumer(ch)
```

**Solution 2: Coordinated Shutdown with WaitGroup**

```go
// ✅ GOOD - Multiple senders, coordinated close
func startWorkers(resultCh chan<- Result, numWorkers int) {
    var wg sync.WaitGroup
    
    for i := 0; i < numWorkers; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            // Send results
            resultCh <- doWork()
        }()
    }
    
    // Close channel after all senders finish
    go func() {
        wg.Wait()
        close(resultCh)  // Safe: all senders done
    }()
}
```

**Solution 3: Use sync.Once for Safety**

```go
// ✅ GOOD - Safe close with sync.Once
type SafeChannel struct {
    ch       chan int
    once     sync.Once
    sendOnce sync.Once
}

func (sc *SafeChannel) Close() {
    sc.once.Do(func() {
        close(sc.ch)  // Only closes once
    })
}

func (sc *SafeChannel) Send(val int) {
    sc.sendOnce.Do(func() {
        defer sc.Close()
        sc.ch <- val
    })
}
```

**Why these are better**:
- Sender controls channel lifecycle
- No panic from multiple closes
- Clear ownership model
- Predictable behavior

### Real-World Panic Examples

**Example 1: HTTP Response Consumer**

```go
// ❌ BAD - Consumer closes channel
func fetchURLs(urls []string) {
    resultCh := make(chan string)
    
    // Start fetchers (senders)
    for _, url := range urls {
        go func(u string) {
            resp, _ := http.Get(u)
            resultCh <- resp.Status
        }(url)
    }
    
    // Consumer
    for i := 0; i < len(urls); i++ {
        fmt.Println(<-resultCh)
    }
    
    close(resultCh)  // PANIC! Senders might still be writing
}
```

```go
// ✅ GOOD - Senders coordinate close
func fetchURLs(urls []string) {
    resultCh := make(chan string, len(urls))
    var wg sync.WaitGroup
    
    // Start fetchers
    for _, url := range urls {
        wg.Add(1)
        go func(u string) {
            defer wg.Done()
            resp, _ := http.Get(u)
            resultCh <- resp.Status
        }(url)
    }
    
    // Close after all senders done
    go func() {
        wg.Wait()
        close(resultCh)  // Safe!
    }()
    
    // Consumer
    for result := range resultCh {
        fmt.Println(result)
    }
}
```

**Example 2: Pipeline with Multiple Stages**

```go
// ❌ BAD - Middle stage closes input
func process(in <-chan int, out chan<- int) {
    defer close(in)  // COMPILE ERROR! Can't close receive-only
    defer close(out)  // OK
    
    for val := range in {
        out <- val * 2
    }
}
```

```go
// ✅ GOOD - Each stage closes its output only
func stage1(out chan<- int) {
    defer close(out)  // Stage 1 closes its output
    for i := 0; i < 5; i++ {
        out <- i
    }
}

func stage2(in <-chan int, out chan<- int) {
    defer close(out)  // Stage 2 closes its output
    for val := range in {
        out <- val * 2
    }
}

func pipeline() {
    ch1 := make(chan int)
    ch2 := make(chan int)
    
    go stage1(ch1)     // Closes ch1
    go stage2(ch1, ch2)  // Closes ch2
    
    for val := range ch2 {
        fmt.Println(val)
    }
}
```

### Panic Scenarios

**Panic 1: Close of Closed Channel**

```go
ch := make(chan int)
close(ch)
close(ch)  // panic: close of closed channel
```

**Panic 2: Send to Closed Channel**

```go
ch := make(chan int)
close(ch)
ch <- 42  // panic: send on closed channel
```

**Panic 3: Close Receive-Only Channel**

```go
func bad(ch <-chan int) {
    close(ch)  // panic: close of receive-only channel
}
```

### Preventing Panics

**Check if closed (receive)**:

```go
// Safe receive with closed check
val, ok := <-ch
if !ok {
    // Channel is closed
    return
}
```

**Use select for safe operations**:

```go
select {
case ch <- val:
    // Sent successfully
case <-time.After(timeout):
    // Timeout, channel might be closed
}
```

**Recover from close panic** (last resort):

```go
func safeClose(ch chan int) (closed bool) {
    defer func() {
        if recover() != nil {
            closed = true
        }
    }()
    
    close(ch)
    return false
}
```

### Channel Lifecycle

```go
// Proper lifecycle management
func workerPool(ctx context.Context) {
    taskCh := make(chan Task, 100)
    resultCh := make(chan Result, 100)
    
    // Producer closes taskCh
    go func() {
        defer close(taskCh)
        for {
            select {
            case task := <-getTask():
                taskCh <- task
            case <-ctx.Done():
                return  // Exit, defer closes channel
            }
        }
    }()
    
    // Workers process tasks
    var wg sync.WaitGroup
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for task := range taskCh {  // Exits when taskCh closes
                resultCh <- process(task)
            }
        }()
    }
    
    // Close resultCh after all workers done
    go func() {
        wg.Wait()
        close(resultCh)
    }()
    
    // Consumer
    for result := range resultCh {
        handleResult(result)
    }
}
```

### Additional Context

**Multiple senders pattern**:

```go
// When multiple goroutines send, use WaitGroup
var wg sync.WaitGroup
ch := make(chan int)

for i := 0; i < 5; i++ {
    wg.Add(1)
    go func() {
        defer wg.Done()
        ch <- compute()
    }()
}

// Coordinator closes channel
go func() {
    wg.Wait()
    close(ch)
}()
```

**Done channel pattern**:

```go
// Sender closes done channel to signal completion
done := make(chan struct{})

go func() {
    defer close(done)  // Signal done
    doWork()
}()

<-done  // Wait for completion
```

### Quick Decision Tree

**Who creates the channel?**
- Parent → **Parent closes (or delegates to sender)**
- Function → **Function closes before returning**

**Multiple senders?**
- Yes → **Use WaitGroup + coordinator to close**
- No → **Single sender closes with defer**

**Receive-only parameter?**
- Yes → **DON'T close it!**
- Send-only → **Close with defer**

**Checklist**:
- [ ] Only sender closes channel
- [ ] Use `defer close()` in sender
- [ ] Receive-only channels never closed
- [ ] Multiple senders use WaitGroup
- [ ] No multiple close() calls

### References

- "Concurrency in Go" by Katherine Cox-Buday - Chapter 3: Channels
- [Go Blog: Share Memory By Communicating](https://go.dev/blog/codelab-share)
- [Effective Go: Channels](https://go.dev/doc/effective_go#channels)

---

**Rule Version**: 1.0  
**Last Updated**: 2026-01-21  
**Priority**: CRITICAL  
**Auto-fixable**: Partially (linter can detect close on receive-only)
