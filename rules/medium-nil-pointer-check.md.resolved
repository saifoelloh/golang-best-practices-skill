---
title: Use Zero Values and Nil Checks
impact: MEDIUM  
impactDescription: Nil pointer panics crash the application
tags: nil, pointer, zero-value, defensive-programming
source: Learning Go - Chapter 6 (Pointers)
---

## Use Zero Values and Nil Checks

Always check pointers for nil before dereferencing. Prefer zero values over pointers for optional fields when possible.

**Impact**: MEDIUM - Runtime panics

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - No nil check
func ProcessUser(user *User) string {
    return user.Name  // Panic if user is nil!
}
```

### Correct (Best Practice)

```go
// ✅ GOOD - Nil check
func ProcessUser(user *User) string {
    if user == nil {
        return ""
    }
    return user.Name
}

// ✅ BETTER - Use zero values
type Config struct {
    Timeout time.Duration  // Zero value is 0, not nil
    Retries int           // Zero value is 0
}
```

---

**Rule Version**: 1.0  
**Priority**: MEDIUM
