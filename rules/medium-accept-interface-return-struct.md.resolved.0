---
title: Accept Interfaces, Return Structs
impact: MEDIUM
impactDescription: Returning interfaces limits evolution and adds unnecessary abstraction
tags: interface, api-design, flexibility, concrete-types
source: Learning Go - Chapter 7 (Interfaces)
---

## Accept Interfaces, Return Structs

Functions should accept interfaces but return concrete types. This maximizes flexibility for callers while maintaining evolution freedom for implementations.

**Impact**: MEDIUM - Limits API evolution, unnecessary abstraction

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Returns interface
func NewUser(name string) UserInterface {
    return &User{Name: name}
}
```

### Correct (Best Practice)

```go
// ✅ GOOD - Returns concrete type
func NewUser(name string) *User {
    return &User{Name: name}
}

// ✅ GOOD - Accepts interface
func SaveUser(repo UserRepository, user *User) error {
    return repo.Save(user)
}
```

---

**Rule Version**: 1.0  
**Priority**: MEDIUM
