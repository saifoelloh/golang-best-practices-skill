---
title: Use Builder Pattern for Complex Construction
impact: MEDIUM
impactDescription: Readability, flexibility, fluent API
tags: design-patterns, builder, fluent-api
source: Design Patterns (GoF); Hands-On Patterns with Go
---

## Use Builder Pattern for Complex Construction

When objects have many optional fields or complex initialization, builder pattern provides fluent, readable construction.

**Impact**: MEDIUM - Improves readability, enables optional configuration

### Detection

When to use builder:
- Constructors with >5 parameters
- Many optional fields
- Complex initialization logic
- Step-by-step construction

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Hard to use, unclear which is which
calculator := NewFeeCalculator(event, ticket, payment, order, groupTicket, partner)
```

**Why this is wrong**:
- Hard to remember order
- Optional params unclear
- Not extensible

### Correct (Best Practice)

```go
// ✅ GOOD - Fluent builder pattern
calculator := NewFeeCalculator(event, ticket, payment, order).
    WithGroupTicket(groupTicket).
    WithPartnerDiscount(partner).
    Calculate()
```

**Why this is better**:
- Self-documenting
- Optional fields clear
- Easy to extend
- Fluent, readable

### References

- [Refactoring.Guru - Builder Pattern](https://refactoring.guru/design-patterns/builder)
- Design Patterns (Gang of Four)
- Hands-On Design Patterns with Go

---

**Rule Version**: 1.0  
**Last Updated**: 2026-01-22
