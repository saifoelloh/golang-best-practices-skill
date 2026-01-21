---
title: Choose Appropriate Buffer Size for Channels
impact: MEDIUM
impactDescription: Wrong buffer size causes blocking or memory waste
tags: channel, buffer, performance, concurrency
source: Concurrency in Go - Chapter 3 (Channels)
---

## Choose Appropriate Buffer Size for Channels

Buffer size should match usage pattern: 1 for signals, num_workers for queues, batch_size for batching. Wrong size causes blocking or wastes memory.

**Impact**: MEDIUM - Performance issues or resource waste

### Guidelines

```go
// ✅ Size 1: Signals/notifications
done := make(chan struct{}, 1)

// ✅ Size = workers: Work distribution
tasks := make(chan Task, 10)  // 10 workers

// ✅ Size = batch: Batch processing
records := make(chan Record, 1000)  // Process in batches of 1000

// ❌ BAD: No buffer when needed
ch := make(chan int)  // Blocks sender!

// ❌ BAD: Huge buffer unnecessarily
ch := make(chan int, 1000000)  // Wastes memory!
```

---

**Rule Version**: 1.0  
**Priority**: MEDIUM
