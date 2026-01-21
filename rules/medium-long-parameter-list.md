---
title: Replace Long Parameter Lists with Structs
impact: MEDIUM
impactDescription: Improves readability, flexibility, IDE support
tags: refactoring, long-parameter-list, parameter-object
source: Refactoring (M. Fowler) - Ch 3; Clean Code (R. Martin) - Ch 3
---

## Replace Long Parameter Lists with Structs

Functions with >5 parameters are hard to remember and error-prone. Parameter objects group related data and improve flexibility.

**Impact**: MEDIUM - Improves readability, reduces errors, enables evolution

### Detection

When to replace:
- Functions with >5 parameters
- Same parameters passed together repeatedly
- Optional parameters using pointers
- Hard to remember parameter order

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - 8 parameters!
func CreateEvent(name, desc, location, startDate, endDate string,
                 price int64, tax *int64, adminFee *int64) error {
    // Which order? Easy to swap startDate/endDate
}
```

**Why this is wrong**:
- Hard to remember order
- Error-prone
- Hard to add new parameters
- Difficult to test

### Correct (Best Practice)

```go
// ✅ GOOD - Parameter object
type CreateEventParams struct {
    Name      string
    Desc      string
    Location  string
    StartDate string
    EndDate   string
    Price     int64
    Tax       *int64
    AdminFee  *int64
}

func CreateEvent(params CreateEventParams) error {
    // Clear, extensible, IDE autocomplete works
}
```

**Why this is better**:
- Self-documenting
- Easy to extend
- IDE support
- Validation can be in struct

### References

- [Refactoring.Guru - Long Parameter List](https://refactoring.guru/smells/long-parameter-list)
- [Refactoring.Guru - Introduce Parameter Object](https://refactoring.guru/introduce-parameter-object)
- Refactoring (Fowler) - Ch 3
- Clean Code (Martin) - Ch 3

---

**Rule Version**: 1.0  
**Last Updated**: 2026-01-22
