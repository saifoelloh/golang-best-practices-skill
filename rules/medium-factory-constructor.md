---
title: Use Factory Functions for Validated Creation
impact: MEDIUM
impactDescription: Encapsulation, validation, clear construction
tags: design-patterns, factory, constructor
source: Design Patterns (GoF); Effective Go
---

## Use Factory Functions for Validated Creation

Factory functions encapsulate creation logic, validation, and initialization in one place. Use `New` prefix by convention.

**Impact**: MEDIUM - Centralizes validation, prevents invalid objects

### Detection

When to use factory:
- Complex creation logic
- Validation required
- Multiple creation paths
- Default initialization needed

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Manual construction, no validation
event := &Event{
    Id: uuid.New().String(),
    Status: "draft", // Magic string
    CreatedAt: time.Now(),
}
```

**Why this is wrong**:
- No validation
- Scattered initialization
- Magic values
- Error-prone

### Correct (Best Practice)

```go
// ✅ GOOD - Factory with validation
func NewDraftEvent(name, organizerId string) (*Event, error) {
    if name == "" {
        return nil, errors.New("name required")
    }
    return &Event{
        Id:          uuid.New().String(),
        Name:        name,
        OrganizerId: organizerId,
        Status:      EventStatusDraft,
        CreatedAt:   time.Now(),
    }, nil
}
```

**Why this is better**:
- Validation centralized
- Cannot create invalid object
- Clear initialization
- Type-safe defaults

### References

- [Refactoring.Guru - Factory Method](https://refactoring.guru/design-patterns/factory-method)
- Design Patterns (Gang of Four)
- Effective Go

---

**Rule Version**: 1.0  
**Last Updated**: 2026-01-22
