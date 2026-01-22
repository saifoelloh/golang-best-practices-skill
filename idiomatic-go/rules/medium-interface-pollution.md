---
title: Keep Interfaces Small (Interface Segregation)
impact: MEDIUM
impactDescription: Large interfaces force implementations to stub many methods
tags: interface, design, ISP, interface-segregation, testability
source: Clean Architecture - Chapter 10 (ISP)
---

## Keep Interfaces Small (Interface Segregation)

Interfaces should have fewer than 5 methods. Small, focused interfaces are easier to implement, test, and maintain. Large interfaces violate the Interface Segregation Principle.

**Impact**: MEDIUM - Harder to test, more coupling, rigid design

### Detection

- Interfaces with >5 methods
- Mocks requiring many stub implementations
- Pattern: `type Service interface { /* 10+ methods */ }`

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Fat interface (10 methods)
type UserService interface {
    CreateUser(user *User) error
    UpdateUser(user *User) error
    DeleteUser(id string) error
    GetUser(id string) (*User, error)
    ListUsers() ([]*User, error)
    AuthenticateUser(email, password string) (*User, error)
    ResetPassword(email string) error
    VerifyEmail(token string) error
    UpdateProfile(user *User) error
    ChangePassword(id, oldPass, newPass string) error
}
```

### Correct (Best Practice)

```go
// ✅ GOOD - Split into focused interfaces
type UserRepository interface {
    Create(user *User) error
    Update(user *User) error
    Delete(id string) error
    GetByID(id string) (*User, error)
    List() ([]*User, error)
}

type UserAuthenticator interface {
    Authenticate(email, password string) (*User, error)
    ResetPassword(email string) error
}

type UserVerifier interface {
    VerifyEmail(token string) error
}
```

---

**Rule Version**: 1.0  
**Priority**: MEDIUM
