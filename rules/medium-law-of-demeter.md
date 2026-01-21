---
title: Law of Demeter - Don't Talk to Strangers
impact: MEDIUM
impactDescription: Reduces coupling, improves encapsulation
tags: refactoring, law-of-demeter, coupling, encapsulation
source: Refactoring.Guru; Clean Code (R. Martin)
---

## Law of Demeter - Don't Talk to Strangers

Avoid calling methods on objects returned by other methods. This creates tight coupling and fragile code.

**Impact**: MEDIUM - Reduces coupling, improves maintainability

### Detection

Message chain signs:
- Chained calls: `a.b().c().d()`
- Reaching through objects
- Accessing nested fields
- Tight coupling

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Message chain, tight coupling
func GetOrganizerEmail(order *Order) string {
    return order.Event.Organizer.User.Email
}

// ❌ BAD - Violates Law of Demeter
func ProcessOrder(order *Order) {
    if order.Payment.Method.Provider.IsActive {
        // Deep navigation
    }
}
```

**Why this is wrong**:
- Changes to intermediate objects break code
- Tight coupling
- Violates encapsulation
- Hard to test

### Correct (Best Practice)

```go
// ✅ GOOD - Delegate through methods
func GetOrganizerEmail(order *Order) string {
    return order.GetOrganizerEmail()
}

func (o *Order) GetOrganizerEmail() string {
    return o.Event.GetOrganizerEmail()
}

func (e *Event) GetOrganizerEmail() string {
    return e.Organizer.GetEmail()
}
```

**Why this is better**:
- Loose coupling
- Changes isolated
- Clear delegation
- Testable

### References

- [Refactoring.Guru - Message Chains](https://refactoring.guru/smells/message-chains)
- [Refactoring.Guru - Inappropriate Intimacy](https://refactoring.guru/smells/inappropriate-intimacy)
- Clean Code (Robert C. Martin)

---

**Rule Version**: 1.0  
**Last Updated**: 2026-01-22
