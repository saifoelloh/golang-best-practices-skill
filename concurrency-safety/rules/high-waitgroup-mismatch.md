---
title: Match WaitGroup Add() and Done() Calls
impact: HIGH
impactDescription: Mismatched calls cause deadlocks or early returns
tags: sync, waitgroup, goroutine, synchronization, deadlock
source: Concurrency in Go - Chapter 2 (Goroutines)
---

## Match WaitGroup Add() and Done() Calls

Every `wg.Add(1)` must have exactly one corresponding `wg.Done()`. Mismatches cause deadlocks or premature completion.

**Impact**: HIGH - Deadlocks or incomplete execution

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Add() inside goroutine (race!)
var wg sync.WaitGroup
for i := 0; i < 5; i++ {
    go func() {
        wg.Add(1)  // Race! May not execute before Wait()
        defer wg.Done()
        doWork()
    }()
}
wg.Wait()  // May return early!
```

### Correct (Best Practice)

```go
// ✅ GOOD - Add() before goroutine
var wg sync.WaitGroup
for i := 0; i < 5; i++ {
    wg.Add(1)  // Before launching goroutine
    go func() {
        defer wg.Done()  // Always matches Add
        doWork()
    }()
}
wg.Wait()  // Waits for all 5
```

---

**Rule Version**: 1.0  
**Priority**: HIGH
