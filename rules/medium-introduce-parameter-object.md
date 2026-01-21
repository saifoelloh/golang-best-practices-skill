---
title: Introduce Parameter Object for Related Data
impact: MEDIUM
impactDescription: Groups related data, reveals domain concepts
tags: refactoring, parameter-object, domain-modeling
source: Refactoring (M. Fowler) - Ch 10
---

## Introduce Parameter Object for Related Data

When several parameters are passed together, they likely represent a cohesive concept. Group them into a struct.

**Impact**: MEDIUM - Clearer domain model, easier to extend

### Detection

When to introduce:
- 3+ parameters passed together
- Parameters represent single concept
- Parameters always used together

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Related params separate
func CalculateFee(basePrice, adminFee, taxFee, paymentFee int64) int64
```

**Why this is wrong**:
- Missing domain concept
- Hard to extend
- Parameter order matters

### Correct (Best Practice)

```go
// ✅ GOOD - Parameter object
type FeeComponents struct {
    BasePrice    int64
    AdminFee     int64
    TaxFee       int64
    PaymentFee   int64
}

func CalculateFee(components FeeComponents) int64
```

**Why this is better**:
- Clear domain concept
- Easy to extend
- Self-documenting

### References

- [Refactoring.Guru - Introduce Parameter Object](https://refactoring.guru/introduce-parameter-object)
- Refactoring (Fowler) - Ch 10

---

**Rule Version**: 1.0  
**Last Updated**: 2026-01-22
