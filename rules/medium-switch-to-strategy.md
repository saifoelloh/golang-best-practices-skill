---
title: Replace Type Switch with Strategy Pattern
impact: MEDIUM
impactDescription: Ext

ensibility, testability, polymorphism
tags: design-patterns, strategy, switch-statements, polymorphism
source: Refactoring.Guru; Design Patterns (GoF)
---

## Replace Type Switch with Strategy Pattern

Type switches (`switch v.(type)`) become unmaintainable as cases grow. Strategy pattern using interfaces provides extensibility and testability.

**Impact**: MEDIUM - Easier to extend, test individual strategies

### Detection

When to replace:
- Type switch with >3 cases
- Same switch pattern repeated
- Adding new types requires changing switch
- Each case has complex logic

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Type switch, hard to extend
func CalculateFee(method PaymentMethod, price int64) int64 {
    switch method.Type {
    case "va":
        return method.FixedFee
    case "qris":
        return int64(float64(price) * method.Rate / 100)
    case "card":
        return int64(float64(price) * method.Rate / 100) + method.FixedFee
    default:
        return 0
    }
}
```

**Why this is wrong**:
- Adding new payment method requires modifying function
- Cannot test strategies independently
- Violates Open/Closed Principle

### Correct (Best Practice)

```go
// ✅ GOOD - Strategy pattern
type FeeStrategy interface {
    Calculate(price int64) int64
}

type VAFee struct {
    FixedFee int64
}

func (v VAFee) Calculate(price int64) int64 {
    return v.FixedFee
}

type QRISFee struct {
    Rate float64
}

func (q QRISFee) Calculate(price int64) int64 {
    return int64(float64(price) * q.Rate / 100)
}

func CalculateFee(strategy FeeStrategy, price int64) int64 {
    return strategy.Calculate(price)
}
```

**Why this is better**:
- Easy to add new strategies
- Each strategy testable independently
- Follows Open/Closed Principle

### References

- [Refactoring.Guru - Switch Statements](https://refactoring.guru/smells/switch-statements)
- [Refactoring.Guru - Strategy Pattern](https://refactoring.guru/design-patterns/strategy)
- Design Patterns (Gang of Four)

---

**Rule Version**: 1.0  
**Last Updated**: 2026-01-22
