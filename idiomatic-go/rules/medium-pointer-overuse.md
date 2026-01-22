---
title: Don't Overuse Pointers for Small Types
impact: MEDIUM
impactDescription: Unnecessary heap allocations and GC pressure for small values
tags: pointer, performance, memory, value-types
source: Learning Go - Chapter 6 (Pointers)
---

## Don't Overuse Pointers for Small Types

Use value types for small structs (<100 bytes) and primitives. Pointers add heap allocations and GC overhead without benefit for small data.

**Impact**: MEDIUM - Performance overhead, unnecessary complexity

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Pointer to primitive
func Calculate(count *int, rate *float64) *int {
    result := *count * int(*rate)
    return &result  // Heap allocation!
}
```

### Correct (Best Practice)

```go
// ✅ GOOD - Value types for small data
func Calculate(count int, rate float64) int {
    return count * int(rate)  // Stack only
}

// ✅ GOOD - Value for small struct
type Point struct {
    X, Y int  // 16 bytes total
}

func Distance(p1, p2 Point) float64 {
    // Pass by value is fine for small structs
}
```

---

**Rule Version**: 1.0  
**Priority**: MEDIUM
