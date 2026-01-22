---
title: Goroutines Must Have Exit Conditions
impact: CRITICAL
impactDescription: Goroutine leaks consume memory indefinitely, leading to OOM crashes
tags: goroutine, concurrency, leak, context, channel, select
source: Concurrency in Go - Chapter 2 (Goroutines)
---

## Goroutines Must Have Exit Conditions

Every goroutine must have a way to terminate. Goroutines without exit conditions leak, consuming memory indefinitely until the program crashes.

**Impact**: CRITICAL - Memory leaks and eventual out-of-memory crashes

### Detection

How to spot this issue:
- `go func()` with infinite `for` loop and no exit condition
- Channel receive in loop without checking if channel is closed
- No `select` with context cancellation case
- Pattern: `for { <-ch }` without range or done check

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Goroutine leaks if channel never closes
func startWorker(taskCh <-chan Task) {
    go func() {
        for {
            task := <-taskCh  // Blocks forever if channel never closes!
            process(task)
        }
    }()
}

// ❌ BAD - No way to signal goroutine to stop
func monitorMetrics() {
    go func() {
        for {
            metrics := collectMetrics()
            sendToServer(metrics)
            time.Sleep(10 * time.Second)
        }
        // Never exits! Leaks when main program wants to shutdown
    }()
}

// ❌ BAD - Infinite select without done case
func handleEvents(eventCh <-chan Event) {
    go func() {
        for {
            select {
            case event := <-eventCh:
                handleEvent(event)
            }
            // No way to break out of loop!
        }
    }()
}
```

**Why this is wrong**:
- Gorout leaked goroutines never garbage collected
- Memory usage grows unbounded
- Can't gracefully shutdown application
- Production crashes from OOM

### Correct (Best Practice)

**Solution 1: Use `for range` (channel closes)**

```go
// ✅ GOOD - Exits when channel closes
func startWorker(taskCh <-chan Task) {
    go func() {
        for task := range taskCh {  // Exits when taskCh closes
            process(task)
        }
        log.Println("Worker exiting cleanly")
    }()
}

// Caller closes channel to signal shutdown
close(taskCh)
```

**Solution 2: Use context cancellation**

```go
// ✅ GOOD - Context signals goroutine to exit
func monitorMetrics(ctx context.Context) {
    go func() {
        ticker := time.NewTicker(10 * time.Second)
        defer ticker.Stop()
        
        for {
            select {
            case <-ticker.C:
                metrics := collectMetrics()
                sendToServer(metrics)
            case <-ctx.Done():
                log.Println("Metrics monitor shutting down")
                return  // Clean exit!
            }
        }
    }()
}

// Caller cancels context to shutdown
ctx, cancel := context.WithCancel(context.Background())
monitorMetrics(ctx)
// Later...
cancel()  // Goroutine exits
```

**Solution 3: Done channel pattern**

```go
// ✅ GOOD - Done channel signals exit
func handleEvents(eventCh <-chan Event, done <-chan struct{}) {
    go func() {
        for {
            select {
            case event := <-eventCh:
                handleEvent(event)
            case <-done:
                log.Println("Event handler shutting down")
                return  // Clean exit!
            }
        }
    }()
}

// Caller closes done to shutdown
doneCh := make(chan struct{})
handleEvents(eventCh, doneCh)
// Later...
close(doneCh)  // Goroutine exits
```

**Why these are better**:
- Goroutines can be terminated gracefully
- No memory leaks
- Clean application shutdown
- Testable (can wait for goroutine exit)

### Real-World Leak Example

```go
// ❌ BAD - Web server with leaking metrics collector
func main() {
    // Starts goroutine that never stops
    go func() {
        for {
            collectAndSendMetrics()
            time.Sleep(30 * time.Second)
        }
    }()
    
    // Start server
    http.ListenAndServe(":8080", handler)
    
    // On shutdown signal, server stops but goroutine leaks!
}

// After restart cycle:
// 1st run: 1 leaked goroutine
// 2nd run: 2 leaked goroutines
// 3rd run: 3 leaked goroutines
// Eventually: Process manager kills due to memory growth
```

```go
// ✅ GOOD - Clean shutdown
func main() {
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()
    
    // Goroutine monitors context
    go func() {
        ticker := time.NewTicker(30 * time.Second)
        defer ticker.Stop()
        
        for {
            select {
            case <-ticker.C:
                collectAndSendMetrics()
            case <-ctx.Done():
                return  // Exits cleanly
            }
        }
    }()
    
    // Handle shutdown signal
    shutdown := make(chan os.Signal, 1)
    signal.Notify(shutdown, syscall.SIGINT, syscall.SIGTERM)
    
    go http.ListenAndServe(":8080", handler)
    
    <-shutdown
    cancel()  // Stops all goroutines
    time.Sleep(time.Second)  // Grace period
}
```

### Worker Pool Pattern

```go
// ✅ GOOD - Worker pool with clean shutdown
func startWorkerPool(ctx context.Context, taskCh <-chan Task, numWorkers int) *sync.WaitGroup {
    var wg sync.WaitGroup
    
    for i := 0; i < numWorkers; i++ {
        wg.Add(1)
        go func(workerID int) {
            defer wg.Done()
            
            for {
                select {
                case task, ok := <-taskCh:
                    if !ok {
                        // Channel closed
                        log.Printf("Worker %d: channel closed, exiting", workerID)
                        return
                    }
                    process(task)
                    
                case <-ctx.Done():
                    // Context cancelled
                    log.Printf("Worker %d: context cancelled, exiting", workerID)
                    return
                }
            }
        }(i)
    }
    
    return &wg
}

// Usage
ctx, cancel := context.WithCancel(context.Background())
taskCh := make(chan Task, 100)
wg := startWorkerPool(ctx, taskCh, 5)

// Send tasks...

// Shutdown gracefully
close(taskCh)  // or cancel()
wg.Wait()  // All workers exit cleanly
```

### Testing for Leaks

```go
import "go.uber.org/goleak"

func TestNoGoroutineLeak(t *testing.T) {
    defer goleak.VerifyNone(t)
    
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()
    
    taskCh := make(chan Task)
    startWorker(ctx, taskCh)
    
    // Send some tasks
    taskCh <- Task{ID: 1}
    taskCh <- Task{ID: 2}
    
    // Cancel and verify goroutine exits
    cancel()
    time.Sleep(100 * time.Millisecond)
    
    // goleak.VerifyNone fails if goroutine is still running
}
```

### Additional Context

**Common leak scenarios**:

```go
// ❌ Starting goroutine in request handler
func (h *Handler) ProcessRequest(c echo.Context) error {
    go func() {
        // Leaks on every request!
        doBackgroundWork()
    }()
    return c.JSON(200, "OK")
}

// ✅ Use context from request
func (h *Handler) ProcessRequest(c echo.Context) error {
    go func() {
        ctx := c.Request().Context()
        select {
        case <-doBackgroundWork():
        case <-ctx.Done():
            return  // Exits if request cancelled
        }
    }()
    return c.JSON(200, "OK")
}
```

**Long-running services**:

```go
// ✅ Proper service lifecycle
type Service struct {
    ctx    context.Context
    cancel context.CancelFunc
    wg     sync.WaitGroup
}

func NewService() *Service {
    ctx, cancel := context.WithCancel(context.Background())
    return &Service{
        ctx:    ctx,
        cancel: cancel,
    }
}

func (s *Service) Start() {
    s.wg.Add(1)
    go func() {
        defer s.wg.Done()
        ticker := time.NewTicker(time.Minute)
        defer ticker.Stop()
        
        for {
            select {
            case <-ticker.C:
                s.doWork()
            case <-s.ctx.Done():
                return
            }
        }
    }()
}

func (s *Service) Stop() {
    s.cancel()
    s.wg.Wait()
}
```

### Quick Decision Tree

**Am I starting a goroutine?**
- Yes → **Does it have an exit condition?**
  - No → **Add context/done channel**
  - Yes → **Good!**

**Exit condition checklist**:
- [ ] `for range` on channel (closes to exit)
- [ ] `select` with `<-ctx.Done()` case
- [ ] `select` with `<-done` channel case
- [ ] Finite loop with break condition

### References

- "Concurrency in Go" by Katherine Cox-Buday - Chapter 2: Goroutines
- [Go Blog: Concurrency Patterns](https://go.dev/blog/pipelines)
- [uber-go/goleak](https://github.com/uber-go/goleak)
- [Effective Go: Goroutines](https://go.dev/doc/effective_go#goroutines)

---

**Rule Version**: 1.0  
**Last Updated**: 2026-01-21  
**Priority**: CRITICAL  
**Auto-fixable**: No (requires design decision)
