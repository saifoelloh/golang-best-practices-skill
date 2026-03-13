---
title: Use Correct JSONB Operators for PostgreSQL JSON Columns
impact: MEDIUM
impactDescription: Wrong operator causes silent empty results or runtime errors on JSONB columns
tags: postgresql, jsonb, json, operators, containment, query
source: PostgreSQL Docs - JSON Functions and Operators
---

## Use Correct JSONB Operators for PostgreSQL JSON Columns

PostgreSQL JSONB has multiple query operators with distinct semantics. Using `->>` when `@>` is needed (or vice versa) produces either silent empty results or type errors.

**Impact**: MEDIUM - Silent empty results or runtime errors on JSON queries

### Key Operators

| Operator | Returns | Use For |
|---|---|---|
| `->` | JSON | Extract JSON object/array by key |
| `->>` | text | Extract value as text (for scalar comparison) |
| `@>` | boolean | Containment check (JSON contains this subset?) |
| `?` | boolean | Does key exist? |

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - ->> extracts as text, can't compare to JSON object
db.Raw("SELECT * FROM orders WHERE metadata->>'items' = ?", items)

// ❌ BAD - comparing extracted text to an integer
db.Raw("SELECT * FROM orders WHERE metadata->>'count' = ?", 5)
// Should cast: metadata->>'count' = ?::text or (metadata->>'count')::int = ?
```

### Correct (Best Practice)

```go
// ✅ GOOD - containment check with @>
db.Raw("SELECT * FROM orders WHERE metadata @> ?::jsonb", `{"status":"paid"}`).Scan(&orders)

// ✅ GOOD - scalar text extraction
db.Raw("SELECT * FROM orders WHERE metadata->>'status' = ?", "paid").Scan(&orders)

// ✅ GOOD - cast extracted value for numeric comparison
db.Raw("SELECT * FROM orders WHERE (metadata->>'total')::numeric > ?", 100).Scan(&orders)
```

### References

- [PostgreSQL Docs: JSON Functions and Operators](https://www.postgresql.org/docs/16/functions-json.html)

---

**Rule Version**: 1.0
**Last Updated**: 2026-03-13
**Priority**: MEDIUM
**Auto-fixable**: No
