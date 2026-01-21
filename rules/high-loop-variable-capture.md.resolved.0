---
title: Avoid Loop Variable Capture in Goroutines
impact: HIGH
impactDescription: All goroutines use last iteration value instead of intended value
tags: closure, loop, goroutine, variable-capture, scope
source: Learning Go - Chapter 5 (Functions)
---

## Avoid Loop Variable Capture in Goroutines

Loop variables are shared across iterations. Goroutines capturing them see the final value, not the iteration value.

**Impact**: HIGH - All goroutines process same (last) value

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - All goroutines see last user!
for _, user := range users {
    go func() {
        sendEmail(user)  // All get last user!
    }()
}
```

### Correct (Best Practice)

```go
// ✅ GOOD - Pass as parameter
for _, user := range users {
    go func(u *User) {
        sendEmail(u)  // Gets correct user
    }(user)
}

// ✅ ALSO GOOD - Shadow variable (Go 1.22+)
for _, user := range users {
    user := user  // Creates new variable
    go func() {
        sendEmail(user)
    }()
}
```

---

**Rule Version**: 1.0  
**Priority**: HIGH
