---
title: Use Directional Channels
impact: MEDIUM
impactDescription: Bidirectional channels allow incorrect usage and reduce clarity
tags: channel, direction, type-safety, concurrency
source: Concurrency in Go - Chapter 3 (Channels)
---

## Use Directional Channels

Channel function parameters should specify direction (`chan<-` send-only, `<-chan` receive-only). This prevents misuse at compile time.

**Impact**: MEDIUM - Runtime bugs from incorrect channel usage

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Bidirectional channels
func producer(ch chan int) {
    ch <- 42
    val := <-ch  // Allowed but wrong!
}

func consumer(ch chan int) {
    val := <-ch
    ch <- 99  // Allowed but wrong!
}
```

### Correct (Best Practice)

```go
// ✅ GOOD - Directional channels
func producer(ch chan<- int) {  // Send-only
    ch <- 42
    // val := <-ch  // Compile error! ✅
}

func consumer(ch <-chan int) {  // Receive-only
    val := <-ch
    // ch <- 99  // Compile error! ✅
}
```

---

**Rule Version**: 1.0  
**Priority**: MEDIUM
