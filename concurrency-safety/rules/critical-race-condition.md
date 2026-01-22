---
title: Protect Shared State with Synchronization
impact: CRITICAL
impactDescription: Race conditions cause data corruption, crashes, and unpredictable behavior
tags: race-condition, concurrency, mutex, sync, data-race, atomic
source: Concurrency in Go - Chapter 1 (Race Conditions)
---

## Protect Shared State with Synchronization

When multiple goroutines access shared state, you **must** protect it with synchronization primitives (`sync.Mutex`, `sync.RWMutex`, or channels). Unprotected concurrent access causes race conditions.

**Impact**: CRITICAL - Data corruption, crashes, unpredictable behavior in production

### Detection

How to spot this issue:
- Shared variables accessed by multiple goroutines
- No mutex/channel protection
- Run with `go run -race` to detect
- Pattern: goroutines reading/writing same variable without locks

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Concurrent map access without protection
type Cache struct {
    data map[string]string  // Shared state!
}

func (c *Cache) Set(key, value string) {
    c.data[key] = value  // RACE! Concurrent writes
}

func (c *Cache) Get(key string) string {
    return c.data[key]  // RACE! Concurrent read during write
}

// Usage with race condition
cache := &Cache{data: make(map[string]string)}

go cache.Set("key1", "value1")  // Goroutine 1 writes
go cache.Set("key2", "value2")  // Goroutine 2 writes
go fmt.Println(cache.Get("key1"))  // Goroutine 3 reads

// Result: fatal error: concurrent map writes
// Or: data corruption, wrong values returned
```

**Why this is wrong**:
- Concurrent map access is undefined behavior
- Can crash with "concurrent map writes/reads"
- Data corruption (writes may be lost)
- Non-deterministic failures (works sometimes!)

### Correct (Best Practice)

**Solution 1: Use sync.Mutex**

```go
// ✅ GOOD - Protected with mutex
type Cache struct {
    mu   sync.Mutex
    data map[string]string
}

func (c *Cache) Set(key, value string) {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.data[key] = value
}

func (c *Cache) Get(key string) string {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.data[key]
}
```

**Solution 2: Use sync.RWMutex (if many reads, few writes)**

```go
// ✅ BETTER - RWMutex for read-heavy workloads
type Cache struct {
    mu   sync.RWMutex  // Allows multiple readers
    data map[string]string
}

func (c *Cache) Set(key, value string) {
    c.mu.Lock()  // Exclusive lock for writes
    defer c.mu.Unlock()
    c.data[key] = value
}

func (c *Cache) Get(key string) string {
    c.mu.RLock()  // Shared lock for reads
    defer c.mu.RUnlock()
    return c.data[key]
}
```

**Solution 3: Use sync.Map (for specific use cases)**

```go
// ✅ GOOD - For concurrent key-value store
var cache sync.Map

func Set(key, value string) {
    cache.Store(key, value)  // Thread-safe
}

func Get(key string) (string, bool) {
    val, ok := cache.Load(key)  // Thread-safe
    if !ok {
        return "", false
    }
    return val.(string), true
}
```

**Why these are better**:
- No race conditions
- Data integrity guaranteed
- Predictable behavior
- Safe for concurrent access

### Real-World Race Condition Examples

**Example 1: Counter Race**

```go
// ❌ BAD - Race on counter
type Metrics struct {
    requestCount int64
}

func (m *Metrics) IncrementRequests() {
    m.requestCount++  // RACE! Lost increments
}

// With 1000 concurrent requests:
// Expected: 1000
// Actual: ~800-950 (non-deterministic!)
```

```go
// ✅ GOOD - Atomic increment
import "sync/atomic"

type Metrics struct {
    requestCount int64
}

func (m *Metrics) IncrementRequests() {
    atomic.AddInt64(&m.requestCount, 1)  // Thread-safe
}

// With 1000 concurrent requests:
// Result: Exactly 1000 (always)
```

**Example 2: Slice Append Race**

```go
// ❌ BAD - Concurrent append
type Logger struct {
    logs []string
}

func (l *Logger) Log(msg string) {
    l.logs = append(l.logs, msg)  // RACE! Data corruption
}

// Result: Panics, lost logs, or corrupted slice
```

```go
// ✅ GOOD - Protected append
type Logger struct {
    mu   sync.Mutex
    logs []string
}

func (l *Logger) Log(msg string) {
    l.mu.Lock()
    defer l.mu.Unlock()
    l.logs = append(l.logs, msg)
}
```

**Example 3: Struct Field Race**

```go
// ❌ BAD - Concurrent struct field access
type User struct {
    Name  string
    Email string
}

func updateUser(u *User) {
    go func() {
        u.Name = "Alice"  // RACE!
    }()
    
    go func() {
        u.Email = "alice@example.com"  // RACE!
    }()
}
```

```go
// ✅ GOOD - Protected with mutex
type User struct {
    mu    sync.RWMutex
    name  string
    email string
}

func (u *User) SetName(name string) {
    u.mu.Lock()
    defer u.mu.Unlock()
    u.name = name
}

func (u *User) SetEmail(email string) {
    u.mu.Lock()
    defer u.mu.Unlock()
    u.email = email
}

func (u *User) GetName() string {
    u.mu.RLock()
    defer u.mu.RUnlock()
    return u.name
}
```

### Detecting Race Conditions

**Run with race detector**:

```bash
# Build with race detector
go build -race ./...

# Run tests with race detector
go test -race ./...

# Run application with race detector
go run -race main.go
```

Race detector output:
```
==================
WARNING: DATA RACE
Write at 0x00c000014090 by goroutine 7:
  main.(*Cache).Set()
      /app/cache.go:15 +0x44

Previous read at 0x00c000014090 by goroutine 6:
  main.(*Cache).Get()
      /app/cache.go:20 +0x38
==================
```

### Performance Considerations

```go
// Mutex performance (in order of preference)

// 1. ✅ BEST - No lock needed (immutable)
type Config struct {
    settings map[string]string  // Read-only after init
}

// 2. ✅ GOOD - RWMutex for read-heavy
type Cache struct {
    mu   sync.RWMutex
    data map[string]string
}

// 3. ✅ OK - Regular mutex
type Counter struct {
    mu    sync.Mutex
    count int
}

// 4. ✅ FAST - Atomic for simple types
type Metrics struct {
    count int64  // Use atomic.AddInt64
}
```

### Additional Context

**When to use each**:

- **sync.Mutex**: General purpose, exclusive access
- **sync.RWMutex**: Many readers, few writers
- **sync.Map**: Concurrent key-value store (specific use cases)
- **atomic**: Simple integer operations
- **Channels**: Communication between goroutines

**Common mistake - partial protection**:

```go
// ❌ BAD - Only some methods protected
type Counter struct {
    mu    sync.Mutex
    count int
}

func (c *Counter) Inc() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.count++
}

func (c *Counter) Get() int {
    return c.count  // RACE! Forgot to lock!
}
```

```go
// ✅ GOOD - All access protected
func (c *Counter) Get() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.count
}
```

### Quick Decision Tree

**Is data accessed by multiple goroutines?**
- No → **No synchronization needed**
- Yes → **Is it read-only after initialization?**
  - Yes → **No synchronization needed**
  - No → **Choose protection**:
    - Simple counter → **Use atomic**
    - Map → **Use sync.Mutex or sync.Map**
    - Struct → **Use sync.Mutex**
    - Many reads, few writes → **Use sync.RWMutex**

### Testing for Races

```go
// Always run tests with -race flag
func TestConcurrentAccess(t *testing.T) {
    cache := NewCache()
    
    var wg sync.WaitGroup
    for i := 0; i < 100; i++ {
        wg.Add(1)
        go func(n int) {
            defer wg.Done()
            cache.Set(fmt.Sprintf("key%d", n), "value")
            _ = cache.Get(fmt.Sprintf("key%d", n))
        }(i)
    }
    wg.Wait()
}

// Run: go test -race
// Fails if race conditions exist
```

### References

- [Go Blog: Race Detector](https://go.dev/blog/race-detector)
- [pkg.go.dev/sync](https://pkg.go.dev/sync)
- "Concurrency in Go" by Katherine Cox-Buday - Chapter 1
- [Effective Go: Concurrency](https://go.dev/doc/effective_go#concurrency)

---

**Rule Version**: 1.0  
**Last Updated**: 2026-01-21  
**Priority**: CRITICAL  
**Auto-fixable**: No (requires design decision)
