---
title: Use Sentinel Errors for Stable Categories
impact: MEDIUM
impactDescription: String errors prevent consistent error handling
tags: error-handling, sentinel-error, error-categories, architecture
source: Learning Go - Chapter 8 (Errors)
---

## Use Sentinel Errors for Stable Categories

Define sentinel errors (`var Err* = errors.New()`) for stable error categories like NotFound, Unauthorized. String errors prevent consistent handling across layers.

**Impact**: MEDIUM - Inconsistent error handling, harder to test

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - String errors
func GetUser(id string) (*User, error) {
    if notFound {
        return nil, errors.New("user not found")
    }
    if unauthorized {
        return nil, errors.New("unauthorized")
    }
}

// Handler can't detect error type!
user, err := GetUser(id)
if err != nil {
    // What HTTP status? Can't tell!
}
```

### Correct (Best Practice)

```go
// ✅ GOOD - Sentinel errors
var (
    ErrNotFound     = errors.New("not found")
    ErrUnauthorized = errors.New("unauthorized")
)

func GetUser(id string) (*User, error) {
    if notFound {
        return nil, fmt.Errorf("user %s: %w", id, ErrNotFound)
    }
    if unauthorized {
        return nil, fmt.Errorf("user %s: %w", id, ErrUnauthorized)
    }
}

// Handler can detect error type!
user, err := GetUser(id)
if errors.Is(err, ErrNotFound) {
    return http.StatusNotFound
}
if errors.Is(err, ErrUnauthorized) {
    return http.StatusUnauthorized
}
```

---

**Rule Version**: 1.0  
**Priority**: MEDIUM
