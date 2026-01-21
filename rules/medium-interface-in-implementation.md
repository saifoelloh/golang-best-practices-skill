---
title: Define Interfaces Where They're Used
impact: MEDIUM
impactDescription: Interfaces in wrong package create unnecessary coupling
tags: interface, package-design, dependency-inversion, clean-architecture
source: Clean Architecture - Dependency Inversion
---

## Define Interfaces Where They're Used

Define interfaces in the package that uses them (consumer), not the package that implements them (producer). This inverts dependencies and reduces coupling.

**Impact**: MEDIUM - Tight coupling, harder to test

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Interface in implementation package
// package repository
type UserRepository interface {
    GetByID(id string) (*User, error)
}

type userRepository struct { ... }
```

### Correct (Best Practice)

```go
// ✅ GOOD - Interface in usecase (consumer)
// package usecase
type UserRepository interface {
    GetByID(id string) (*User, error)
}

// package repository
type userRepository struct { ... }

// Implements usecase.UserRepository implicitly
```

---

**Rule Version**: 1.0  
**Priority**: MEDIUM
