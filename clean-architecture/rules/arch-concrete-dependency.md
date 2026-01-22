---
title: Depend on Interfaces, Not Concrete Types
impact: ARCHITECTURE
impactDescription: Enables testability, flexibility, and dependency inversion
tags: clean-architecture, interfaces, dependency-injection
source: Clean Architecture (Robert C. Martin) - Chapter 11
---

## Depend on Interfaces, Not Concrete Types

High-level modules (usecases) should depend on interfaces, not concrete implementations. This enables dependency inversion, testability, and flexibility.

**Impact**: ARCHITECTURE - Tight coupling to concrete types makes code untestable and inflexible.

### Detection

How to spot this issue in code:
- Usecase fields are concrete types (e.g., `*UserRepoImpl`)
- Constructor parameters are concrete types
- Function parameters are struct pointers instead of interfaces
- Direct instantiation of dependencies

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Usecase depending on concrete repository
package usecase

import "myapp/repository"

type UserUsecase struct {
    userRepo *repository.UserRepoImpl // BAD: concrete type dependency
}

func NewUserUsecase() *UserUsecase {
    return &UserUsecase{
        userRepo: &repository.UserRepoImpl{}, // BAD: creating concrete dependency
    }
}

func (u *UserUsecase) GetUser(id int64) (*domain.User, error) {
    return u.userRepo.FindByID(id) // Tightly coupled to concrete implementation
}
```

**Why this is wrong**:
- Cannot mock repository for testing
- Cannot swap implementations
- Violates dependency inversion principle
- Hard coupling between layers

### Correct (Best Practice)

```go
// ✅ GOOD - Domain defines interface
package domain

type UserRepository interface {
    FindByID(ctx context.Context, id int64) (*User, error)
    Save(ctx context.Context, user *User) error
}
```

```go
// ✅ GOOD - Usecase depends on interface
package usecase

import "myapp/domain"

type UserUsecase struct {
    userRepo domain.UserRepository // GOOD: interface dependency
}

func NewUserUsecase(userRepo domain.UserRepository) *UserUsecase {
    return &UserUsecase{
        userRepo: userRepo, // GOOD: injected dependency
    }
}

func (u *UserUsecase) GetUser(ctx context.Context, id int64) (*domain.User, error) {
    return u.userRepo.FindByID(ctx, id) // Works with any implementation
}
```

```go
// ✅ GOOD - Test with mock
func TestUserUsecase_GetUser(t *testing.T) {
    mockRepo := &MockUserRepo{} // Mock implements domain.UserRepository
    usecase := NewUserUsecase(mockRepo)
    
    user, err := usecase.GetUser(context.Background(), 1)
    // Test logic
}
```

**Why this is better**:
- Easy to test with mocks
- Can swap implementations without changing usecase
- Follows dependency inversion principle
- Loose coupling between layers

### Additional Context

**Dependency Inversion Principle (DIP)**:
> High-level modules should not depend on low-level modules. Both should depend on abstractions.

**Where to define interfaces**:
- In the domain layer (consumed by usecases)
- In the package that USES the interface (consumer-defined)
- NOT in the package that implements it

**Example folder structure**:
```
domain/
  user.go              # User entity
  user_repository.go   # UserRepository interface (defined by consumer)

usecase/
  user_usecase.go      # Uses domain.UserRepository interface

repository/
  user_repo_impl.go    # Implements domain.UserRepository
```

**Related rules**:
- `high-constructor-creates-deps` - Inject dependencies, don't create
- `arch-interface-segregation` - Keep interfaces small

### References

- [Clean Architecture (Robert C. Martin) - Dependency Inversion](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Go Proverbs: "Accept interfaces, return structs"](https://www.youtube.com/watch?v=PAAkCSZUG1c)
- [Effective Go - Interfaces](https://go.dev/doc/effective_go#interfaces)

---

**Rule Template Version**: 1.0  
**Last Updated**: 2026-01-22
