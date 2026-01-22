---
title: Interface Segregation - Small, Consumer-Defined Interfaces
impact: ARCHITECTURE
impactDescription: Prevents forced dependencies on unused methods, improves testability
tags: clean-architecture, interface-segregation, solid-principles
source: Clean Architecture (Robert C. Martin) - Chapter 10, Learning Go - Chapter 7
---

## Interface Segregation - Small, Consumer-Defined Interfaces

Interfaces should be small (1-3 methods) and defined where they are USED, not where they are implemented. Clients should not depend on methods they don't use.

**Impact**: ARCHITECTURE - Large interfaces force implementations to satisfy methods they don't need, making code harder to test and maintain.

### Detection

How to spot this issue in code:
- Interface with more than 5 methods
- Mock implementations with empty/panic methods
- Interface defined in same package as implementation
- Clients depending on "fat" interfaces but using only 1-2 methods
- Repository interfaces with 20+ methods

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Fat interface defined in repository package
package repository

// BAD: Too many methods, violates ISP
type UserRepository interface {
    Create(ctx context.Context, user *domain.User) error
    Update(ctx context.Context, user *domain.User) error
    Delete(ctx context.Context, id int64) error
    FindByID(ctx context.Context, id int64) (*domain.User, error)
    FindByEmail(ctx context.Context, email string) (*domain.User, error)
    FindByUsername(ctx context.Context, username string) (*domain.User, error)
    FindAll(ctx context.Context) ([]*domain.User, error)
    FindPaginated(ctx context.Context, offset, limit int) ([]*domain.User, error)
    FindActive(ctx context.Context) ([]*domain.User, error)
    FindInactive(ctx context.Context) ([]*domain.User, error)
    CountAll(ctx context.Context) (int64, error)
    CountActive(ctx context.Context) (int64, error)
    UpdatePassword(ctx context.Context, id int64, hash string) error
    UpdateLastLogin(ctx context.Context, id int64, time time.Time) error
    // ... 20+ methods
}
```

```go
// ❌ BAD - Usecase forced to depend on all methods
package usecase

type LoginUsecase struct {
    userRepo repository.UserRepository // BAD: depends on 20+ methods but uses only 2
}

func (u *LoginUsecase) Login(ctx context.Context, email string) error {
    user, _ := u.userRepo.FindByEmail(ctx, email)      // Only uses this
    _ = u.userRepo.UpdateLastLogin(ctx, user.ID, time.Now()) // And this
    
    // Doesn't use other 18+ methods, but forced to mock them all in tests
    return nil
}
```

**Why this is wrong**:
- LoginUsecase depends on methods it doesn't use (ISP violation)
- Testing requires mocking 20+ methods
- Changes to unused methods break unrelated code
- Interface defined by implementer, not consumer

### Correct (Best Practice)

```go
// ✅ GOOD - Small, consumer-defined interfaces in usecase package
package usecase

// GOOD: Small interface, defined where it's used
type UserEmailFinder interface {
    FindByEmail(ctx context.Context, email string) (*domain.User, error)
}

type UserLoginUpdater interface {
    UpdateLastLogin(ctx context.Context, id int64, time time.Time) error
}

type LoginUsecase struct {
    userFinder UserEmailFinder   // GOOD: Only depends on what it needs
    loginUpdater UserLoginUpdater
}

func (u *LoginUsecase) Login(ctx context.Context, email string) error {
    user, err := u.userFinder.FindByEmail(ctx, email)
    if err != nil {
        return err
    }
    
    return u.loginUpdater.UpdateLastLogin(ctx, user.ID, time.Now())
}
```

```go
// ✅ GOOD - Repository implements multiple small interfaces
package repository

type UserRepoImpl struct {
    db *gorm.DB
}

// Automatically satisfies usecase.UserEmailFinder
func (r *UserRepoImpl) FindByEmail(ctx context.Context, email string) (*domain.User, error) {
    var user domain.User
    err := r.db.Where("email = ?", email).First(&user).Error
    return &user, err
}

// Automatically satisfies usecase.UserLoginUpdater
func (r *UserRepoImpl) UpdateLastLogin(ctx context.Context, id int64, t time.Time) error {
    return r.db.Model(&domain.User{}).Where("id = ?", id).
        Update("last_login", t).Error
}

// ... other methods that satisfy other consumer interfaces
```

```go
// ✅ GOOD - Testing is simple (only mock what you need)
package usecase_test

type MockUserFinder struct {
    FindByEmailFunc func(ctx context.Context, email string) (*domain.User, error)
}

func (m *MockUserFinder) FindByEmail(ctx context.Context, email string) (*domain.User, error) {
    return m.FindByEmailFunc(ctx, email)
}

func TestLoginUsecase_Login(t *testing.T) {
    mockFinder := &MockUserFinder{
        FindByEmailFunc: func(ctx context.Context, email string) (*domain.User, error) {
            return &domain.User{ID: 1, Email: email}, nil
        },
    }
    
    // GOOD: Only need to mock 1 method, not 20+
    usecase := NewLoginUsecase(mockFinder, mockUpdater)
    // ... test
}
```

**Why this is better**:
- Each consumer defines exactly what it needs
- Easy to test (small mocks)
- Changes to other methods don't affect unrelated code
- Follows Interface Segregation Principle
- Implements "Accept interfaces, return structs" Go proverb

### Additional Context

**Interface Segregation Principle (ISP)**:
> No client should be forced to depend on methods it does not use.

**Where to define interfaces**:
```
domain/
  user.go

usecase/
  login_usecase.go      # Defines UserEmailFinder interface HERE
  profile_usecase.go    # Defines UserProfileFinder interface HERE

repository/
  user_repo_impl.go     # Implements BOTH interfaces (doesn't define them)
```

**When to split interfaces**:
- Interface has >5 methods
- Different consumers use different subsets of methods
- Testing requires mocking many unused methods

**Example: Split large repository**:
```go
// Instead of one fat UserRepository interface with 20 methods:

// Consumer 1 needs only this
type UserCreator interface {
    Create(ctx context.Context, user *domain.User) error
}

// Consumer 2 needs only this
type UserFinder interface {
    FindByID(ctx context.Context, id int64) (*domain.User, error)
}

// Consumer 3 needs only this
type UserDeleter interface {
    Delete(ctx context.Context, id int64) error
}

// Repository implements all three
type UserRepoImpl struct { ... }
// Implements Create, FindByID, Delete
```

**Related rules**:
- `medium-interface-pollution` - Keep interfaces small (<5 methods)
- `arch-concrete-dependency` - Depend on interfaces, not concrete types
- `medium-accept-interface-return-struct` - API flexibility pattern

### References

- [Clean Architecture (Robert C. Martin) - ISP](https://blog.cleancoder.com/uncle-bob/2014/05/08/SingleReponsibilityPrinciple.html)
- [Go Proverbs: "Accept interfaces, return structs"](https://www.youtube.com/watch?v=PAAkCSZUG1c)
- [Interface Segregation Principle](https://en.wikipedia.org/wiki/Interface_segregation_principle)

---

**Rule Template Version**: 1.0  
**Last Updated**: 2026-01-22
