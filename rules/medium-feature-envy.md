---
title: Move Logic Closer to Data
impact: MEDIUM
impactDescription: Encapsulation, cohesion, testability
tags: refactoring, feature-envy, move-method
source: Refactoring (M. Fowler) - Ch 3; Clean Code (R. Martin)
---

## Move Logic Closer to Data

When a method accesses another object's data more than its own, move the logic to where the data lives. This improves encapsulation and cohesion.

**Impact**: MEDIUM - Better encapsulation, clearer responsibilities

### Detection

Feature envy signs:
- Method accesses another object's fields >3 times
- Complex calculations on external data
- Business logic in wrong layer

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Usecase envies domain data
func (u *OrderUsecase) CalculateDiscount(ptt *PartnerTicketType, price int64) int64 {
    discount := ptt.GetDiscount(ticketTypeId)
    if discount.Type == "percentage" {
        return price * discount.Value / 100
    }
    return discount.Value
}
```

**Why this is wrong**:
- Business logic in usecase
- Usecase reaches into domain
- Can't test discount logic without usecase

### Correct (Best Practice)

```go
// ✅ GOOD - Logic moved to domain
func (ptt *PartnerTicketType) CalculateDiscountedPrice(price int64, ticketTypeId string) int64 {
    discount := ptt.GetDiscount(ticketTypeId)
    if discount.Type == "percentage" {
        return price * discount.Value / 100
    }
    return discount.Value
}
```

**Why this is better**:
- Domain logic in domain layer
- Testable independently
- Clear encapsulation

### References

- [Refactoring.Guru - Feature Envy](https://refactoring.guru/smells/feature-envy)
- [Refactoring.Guru - Move Method](https://refactoring.guru/move-method)
- Refactoring (Fowler) - Ch 3

---

**Rule Version**: 1.0  
**Last Updated**: 2026-01-22
