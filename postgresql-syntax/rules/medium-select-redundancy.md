---
title: Avoid Manual Quoting in Simple GORM .Select() Calls
impact: MEDIUM
impactDescription: Manual quotes in simple .Select() are redundant — reduces readability and portability
tags: gorm, select, identifier-quoting, redundancy, readability
source: GORM Docs - Selecting Specific Fields
---

## Avoid Manual Quoting in Simple GORM .Select() Calls

GORM automatically quotes simple column names in `.Select()` using the correct dialect. Manual escaping (`\"column\"`) in simple column lists is redundant, hurts readability, and reduces portability.

**Impact**: MEDIUM - Reduced code readability, harder maintenance

> **Note**: This is the complement to `high-identifier-quoting.md`. That rule covers reserved keywords in raw SQL fragments (where quoting IS required). This rule covers simple field lists in `.Select()` where quoting is NOT needed.

### Detection

How to spot this issue:
- `db.Select("\"id\"", "\"name\"")` — manually escaped simple column names
- `db.Select("\"order\"")` where `order` is a reserved keyword — this is the exception, see note below
- Backtick strings with unnecessary identifier quotes in `.Select()` of plain column names

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - GORM quotes these automatically, manual quoting is redundant
db.Select("\"id\"", "\"name\"", "\"created_at\"").Find(&users)

// ❌ BAD - less readable, no benefit for plain column names
db.Model(&Asset{}).Select("\"id\"", "\"status\"").Find(&assets)
```

### Correct (Best Practice)

```go
// ✅ CORRECT - let GORM handle quoting for plain columns
db.Select("id", "name", "created_at").Find(&users)
db.Model(&Asset{}).Select("id", "status").Find(&assets)

// ✅ CORRECT - manual quotes ARE needed for reserved keywords inside expressions
db.Select(`"order"`, "id", "created_at").Find(&shipments)
//          ^^^^^^^ reserved keyword — must quote manually

// ✅ CORRECT - expressions always need manual quoting if using reserved words
db.Select(`COALESCE("order", 0) as order_num`, "id").Find(&results)
```

**Why this is better**:
- Plain column names: GORM auto-quotes → no manual escaping needed
- Cleaner, more readable code
- Let the library handle dialect differences (PostgreSQL double quotes vs MySQL backticks)

### When Manual Quoting IS Required

| Context | Quote Required? | Example |
|---|---|---|
| Simple `.Select("id")` | ❌ No | `db.Select("id")` |
| Reserved word in `.Select()` | ✅ Yes | `db.Select('"order"')` |
| Reserved word in `.Order()` | ✅ Yes | `db.Order('"order" ASC')` |
| Reserved word in `.Where()` | ✅ Yes | `db.Where('"order" = ?', 1)` |
| Reserved word in `db.Raw()` expression | ✅ Yes | `db.Raw('SELECT "order" FROM ...')` |

### References

- [GORM Docs: Selecting Specific Fields](https://gorm.io/docs/query.html#Selecting-Specific-Fields)

---

**Rule Version**: 1.0
**Last Updated**: 2026-03-13
**Priority**: MEDIUM
**Auto-fixable**: Yes (remove unnecessary quotes from plain column names in .Select())
