---
title: Extract Repeated Parameter Groups
impact: MEDIUM
impactDescription: DRY principle, clear domain concepts
tags: refactoring, data-clumps, parameter-object
source: Refactoring (M. Fowler) - Ch 3
---

## Extract Repeated Parameter Groups

When the same 3+ parameters appear together in multiple functions, they likely represent a missing domain concept. Extract them into a struct.

**Impact**: MEDIUM - Reduces duplication, reveals domain concepts

### Detection

Data clumps signs:
- Same 3+ parameters in 3+ functions
- Parameters always passed together
- Missing cohesive domain name

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Same params everywhere
func GetTickets(eventId, organizerId, userId string) ([]*Ticket, error)
func GetOrders(eventId, organizerId, userId string) ([]*Order, error)
func GetTransactions(eventId, organizerId, userId string) ([]*Tx, error)
```

**Why this is wrong**:
- Duplication
- Missing domain concept
- Change requires updating many signatures

### Correct (Best Practice)

```go
// ✅ GOOD - Extract to struct
type EventContext struct {
    EventID      string
    OrganizerID  string
    UserID       string
}

func GetTickets(ctx EventContext) ([]*Ticket, error)
func GetOrders(ctx EventContext) ([]*Order, error)
func GetTransactions(ctx EventContext) ([]*Tx, error)
```

**Why this is better**:
- Clear domain concept
- DRY
- Easy to extend
- Single place to change

### References

- [Refactoring.Guru - Data Clumps](https://refactoring.guru/smells/data-clumps)
- Refactoring (Fowler) - Ch 3

---

**Rule Version**: 1.0  
**Last Updated**: 2026-01-22
