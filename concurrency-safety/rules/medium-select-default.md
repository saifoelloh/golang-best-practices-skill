---
title: Avoid Busy-Wait with select/default
impact: MEDIUM
impactDescription: Busy-wait consumes 100% CPU doing nothing useful
tags: select, busy-wait, cpu, performance, anti-pattern
source: Concurrency in Go - Chapter 4 (Select)
---

## Avoid Busy-Wait with select/default

Don't use `select` with `default` in a loop for polling. This creates a busy-wait that wastes CPU. Use blocking select or time.Ticker instead.

**Impact**: MEDIUM - 100% CPU usage, battery drain

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Busy-wait (100% CPU!)
for {
    select {
    case val := <-ch:
        process(val)
    default:
        // Spins continuously! CPU maxed out
    }
}
```

### Correct (Best Practice)

```go
// ✅ GOOD - Blocking select
for {
    select {
    case val := <-ch:
        process(val)
    case <-ctx.Done():
        return
    }
}

// ✅ GOOD - Ticker for periodic checks
ticker := time.NewTicker(100 * time.Millisecond)
defer ticker.Stop()
for {
    select {
    case val := <-ch:
        process(val)
    case <-ticker.C:
        checkStatus()
    }
}
```

---

**Rule Version**: 1.0  
**Priority**: MEDIUM
