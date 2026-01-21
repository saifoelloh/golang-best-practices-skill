---
title: Replace Primitives with Value Objects
impact: MEDIUM
impactDescription: Type safety, validation, domain clarity
tags: refactoring, primitive-obsession, value-objects, ddd
source: Refactoring (M. Fowler) - Ch 3; DDD (E. Evans)
---

## Replace Primitives with Value Objects

Using primitives (string, int64) for domain concepts loses type safety and scatters validation. Value objects encapsulate validation and make domain concepts explicit.

**Impact**: MEDIUM - Improves type safety, centralizes validation, clarifies domain

### Detection

Signs of primitive obsession:
- Email, phone, UUID as `string`
- Money as `int64` or `float64`
- Status/enum as `string` 
- Validation scattered across codebase
- Type confusion (passing wrong string type)

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Primitive obsession
type Event struct {
    Email       string // Any string accepted
    PhoneNumber string // No format validation
    Status      string // What are valid values?
}

func UpdateEvent(email string) error {
    if !strings.Contains(email, "@") {
        return errors.New("invalid email")
    }
    // Validation scattered everywhere email is used
}
```

**Why this is wrong**:
- No compile-time safety
- Validation duplicated
- Easy to pass wrong value
- Domain concepts implicit

### Correct (Best Practice)

```go
// ✅ GOOD - Value objects
type Email struct {
    value string
}

func NewEmail(email string) (Email, error) {
    if !isValidEmail(email) {
        return Email{}, errors.New("invalid email")
    }
    return Email{value: email}, nil
}

func (e Email) String() string { return e.value }

type EventStatus string
const (
    EventStatusDraft    EventStatus = "draft"
    EventStatusActive   EventStatus = "active"
    EventStatusArchived EventStatus = "archived"
)

type Event struct {
    Email       Email
    PhoneNumber PhoneNumber
    Status      EventStatus
}
```

**Why this is better**:
- Validation in one place
- Type-safe
- Self-documenting
- Impossible to create invalid value

### References

- [Refactoring.Guru - Primitive Obsession](https://refactoring.guru/smells/primitive-obsession)
- Refactoring (Fowler) - Ch 3
- Domain-Driven Design (Evans)

---

**Rule Version**: 1.0  
**Last Updated**: 2026-01-22
