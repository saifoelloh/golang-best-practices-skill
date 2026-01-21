---
title: Limit Concurrent Goroutines (Worker Pool)
impact: HIGH
impactDescription: Unbounded goroutines exhaust system resources causing crashes
tags: concurrency, goroutine, worker-pool, resource-management, semaphore
source: Concurrency in Go - Chapter 5 (Concurrency at Scale)
---

## Limit Concurrent Goroutines (Worker Pool)

Never create unbounded goroutines in loops. Use worker pools or semaphores to limit concurrency and prevent resource exhaustion.

**Impact**: HIGH - System crashes from too many goroutines/connections

### Detection

- `go func()` inside unbounded loop
- No limit on concurrent operations
- Pattern: `for _, item := range items { go process(item) }`

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Creates millions of goroutines
func ProcessUsers(users []*User) {
    for _, user := range users {  // Could be 1M users!
        go func(u *User) {
            sendEmail(u)  // 1M concurrent email sends!
        }(user)
    }
}
```

###Correct (Best Practice)

```go
// ✅ GOOD - Worker pool with fixed size
func ProcessUsers(users []*User) {
    const numWorkers = 10
    userCh := make(chan *User, 100)
    
    var wg sync.WaitGroup
    for i := 0; i < numWorkers; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for user := range userCh {
                sendEmail(user)
            }
        }()
    }
    
    for _, user := range users {
        userCh <- user
    }
    close(userCh)
    wg.Wait()
}
```

---

**Rule Version**: 1.0  
**Priority**: HIGH
