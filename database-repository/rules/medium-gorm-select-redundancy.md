---
name: medium-gorm-select-redundancy
description: Avoid manual quotes in simple .Select() calls as GORM handles this automatically.
priority: MEDIUM
---

# GORM Select Redundancy (`medium-gorm-select-redundancy`)

## Why it Matters
GORM automatically quotes simple column names in `.Select()` calls using the appropriate dialect-specific quoting. Manual quoting (e.g., using `\"column\"` or backticks) is redundant, less portable, and reduces code readability.

## Detection
- Look for explicit quotes like `\"column\"` or backticks inside simple string arguments to `.Select()`.

## Incorrect Code Example (❌ BAD)
```go
db.Select("\"id\"", "\"order\"").Find(&assets)
```

## Correct Code Example (✅ GOOD)
```go
// GORM handles quoting automatically for these simple identifiers
db.Select("id", "order").Find(&assets)
```

## Impact Assessment
- **Criticality**: MEDIUM
- **Maintainability**: Cleaner, more idiomatic GORM code.
- **Portability**: Allows GORM to handle dialect-specific quoting automatically.

## References
- [GORM Documentation: Query - Selecting Specific Fields](https://gorm.io/docs/query.html#Selecting-Specific-Fields)
