---
title: Inject Dependencies, Don't Create Them
impact: HIGH
impactDescription: Creating dependencies internally makes code untestable and tightly coupled
tags: dependency-injection, clean-architecture, testing, coupling
source: Clean Architecture - Chapter 11 (DIP)
---

## Inject Dependencies, Don't Create Them

Constructors should accept dependencies as parameters, not create them internally. This enables testing and follows Dependency Inversion Principle.

**Impact**: HIGH - Untestable code, tight coupling

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Creates dependencies internally
func NewUserService() *UserService {
    db := postgresql.NewDB()  // Hardcoded!
    return &UserService{db: db}
}
```

### Correct (Best Practice)

```go
// ✅ GOOD - Dependencies injected
func NewUserService(repo UserRepository) *UserService {
    return &UserService{repo: repo}
}
```

---

**Rule Version**: 1.0  
**Priority**: HIGH
