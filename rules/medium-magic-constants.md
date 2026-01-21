---
title: Replace Magic Numbers with Named Constants
impact: MEDIUM
impactDescription: Readability, maintainability, clear intent
tags: refactoring, magic-numbers, constants, readability
source: Refactoring (M. Fowler) - Ch 8; Clean Code (R. Martin) - Ch 17
---

## Replace Magic Numbers with Named Constants

Literal values without context are "magic" - their meaning is unclear. Named constants document intent and centralize configuration.

**Impact**: MEDIUM - Improves readability, eases configuration changes

### Detection

Magic number signs:
- Literals in code (except 0, 1, -1)
- Same number in multiple places
- Unclear meaning from context
- Need comments to explain value

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - What is 2000? 100? 10?
withdrawalFee := int64(2000)
if tax <= 100 {
    // percentage
}
taxFee = basePrice * 10 / 100
```

**Why this is wrong**:
- Unclear meaning
- Hard to change
- Duplicate values
- No documentation

### Correct (Best Practice)

```go
// ✅ GOOD - Named constants
const (
    DefaultWithdrawalFee = 2000
    TaxPercentageThreshold = 100
    DefaultTaxRatePercent = 10
)

withdrawalFee := DefaultWithdrawalFee
if tax <= TaxPercentageThreshold {
    // percentage
}
taxFee = basePrice * DefaultTaxRatePercent / 100
```

**Why this is better**:
- Self-documenting
- Single source of truth
- Easy to change
- Type-safe

### References

- [Refactoring.Guru - Magic Numbers](https://refactoring.guru/replace-magic-number-with-symbolic-constant)
- Refactoring (Fowler) - Ch 8
- Clean Code (Martin) - Ch 17

---

**Rule Version**: 1.0  
**Last Updated**: 2026-01-22
