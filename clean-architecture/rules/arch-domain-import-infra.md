---
title: Domain Must Not Import Infrastructure
impact: ARCHITECTURE
impactDescription: Violates Clean Architecture dependency rule, creates tight coupling
tags: clean-architecture, dependencies, domain-layer
source: Clean Architecture (Robert C. Martin) - Chapter 22
---

## Domain Must Not Import Infrastructure

The domain layer must not import infrastructure packages (database, HTTP, gRPC, external services). Dependencies must point INWARD toward the domain, never outward.

**Impact**: ARCHITECTURE - Violates fundamental Clean Architecture principle, makes domain untestable and tightly coupled to infrastructure choices.

### Detection

How to spot this issue in code:
- Domain package imports database drivers (e.g., `gorm`, `pgx`)
- Domain entities import HTTP/gRPC packages
- Domain interfaces import infrastructure types
- Import cycle involving domain and infrastructure

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Domain entity importing GORM
package domain

import (
    "gorm.io/gorm" // BAD: domain importing infrastructure
)

type User struct {
    gorm.Model        // BAD: infrastructure type in domain
    Name  string
    Email string
}

func (u *User) Save(db *gorm.DB) error { // BAD: infrastructure dependency
    return db.Create(u).Error
}
```

**Why this is wrong**:
- Domain is now coupled to GORM (can't switch databases)
- Domain entities can't be tested without database
- Violates dependency inversion principle
- Makes domain "impure" - contains infrastructure concerns

### Correct (Best Practice)

```go
// ✅ GOOD - Pure domain entity
package domain

import "context"

type User struct {
    ID    int64
    Name  string
    Email string
}

func (u *User) Validate() error {
    // Pure business logic, no infrastructure
    if u.Email == "" {
        return errors.New("email required")
    }
    return nil
}

// Domain defines interface, doesn't import infrastructure
type UserRepository interface {
    Save(ctx context.Context, user *User) error
    FindByID(ctx context.Context, id int64) (*User, error)
}
```

```go
// ✅ GOOD - Repository implementation in infrastructure layer
package repository

import (
    "context"
    "gorm.io/gorm"
    "myapp/domain" // Infrastructure imports domain, not vice versa
)

type UserRepoImpl struct {
    db *gorm.DB
}

func (r *UserRepoImpl) Save(ctx context.Context, user *domain.User) error {
    return r.db.Create(user).Error
}
```

**Why this is better**:
- Domain is pure, no infrastructure dependencies
- Can test domain logic without database
- Easy to swap database implementations
- Follows dependency inversion principle

### Additional Context

**Clean Architecture Layers**:
```
Outer → Inner (dependencies point inward only)
Infrastructure → Usecase → Domain
```

**Allowed domain imports**:
- Standard library (context, errors, time)
- Other domain packages
- Pure utility libraries (no I/O)

**Forbidden domain imports**:
- Database drivers (gorm, pgx, mongo)
- HTTP/gRPC frameworks
- External service clients
- Caching libraries

**Related rules**:
- `arch-concrete-dependency` - Use interfaces, not concrete types
- `arch-interface-segregation` - Keep interfaces small

### References

- [Clean Architecture (Robert C. Martin)](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Hexagonal Architecture](https://alistair.cockburn.us/hexagonal-architecture/)
- [Go Project Layout](https://github.com/golang-standards/project-layout)

---

**Rule Template Version**: 1.0  
**Last Updated**: 2026-01-22
