---
title: Always Close Channels When Done
impact: HIGH
impactDescription: Unclosed channels cause goroutine leaks waiting on range
tags: channel, close, goroutine-leak, concurrency
source: Concurrency in Go - Chapter 3 (Channels)
---

## Always Close Channels When Done

Channels should be closed when no more values will be sent. This signals receivers to stop waiting and prevents goroutine leaks.

**Impact**: HIGH - Goroutines leak waiting for closed channels

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Channel never closed
func producer(ch chan int) {
    for i := 0; i < 10; i++ {
        ch <- i
    }
    // Missing close(ch)!
}

func consumer(ch chan int) {
    for val := range ch {  // Waits forever!
        process(val)
    }
}
```

### Correct (Best Practice)

```go
// ✅ GOOD - Producer closes channel
func producer(ch chan int) {
    defer close(ch)  // Close when done
    for i := 0; i < 10; i++ {
        ch <- i
    }
}

func consumer(ch chan int) {
    for val := range ch {  // Exits when closed
        process(val)
    }
}
```

---

**Rule Version**: 1.0  
**Priority**: HIGH
