---
name: high-gorm-identifier-quoting
description: Quote reserved keywords in raw SQL fragments to ensure PostgreSQL compatibility.
priority: HIGH
---

# GORM Identifier Quoting (`high-gorm-identifier-quoting`)

## Why it Matters
GORM does not automatically quote identifiers in raw SQL fragments like `.Where()`, `.Order()`, or complex expressions. Reserved keywords like `order`, `user`, `group`, or `limit` will cause syntax errors in PostgreSQL if unquoted.

## Detection
- Look for reserved words (`order`, `user`, `group`, `limit`) in string arguments of `.Order()`, `.Where()`, or inside `gorm.Expr()`.

## Incorrect Code Example (❌ BAD)
```go
db.Order("order ASC").Find(&results)
db.Where("order = ?", 1).First(&result)
db.Select("COALESCE(MAX(order), 0)")
```

## Correct Code Example (✅ GOOD)
```go
// Use double quotes (escaped) for PostgreSQL standard
db.Order("\"order\" ASC").Find(&results)
db.Where("\"order\" = ?", 1).First(&result)
db.Select("COALESCE(MAX(\"order\"), 0)")
```

## Impact Assessment
- **Criticality**: HIGH
- **Reliability**: Ensures queries don't fail in production due to reserved keyword conflicts.
- **Portability**: Improves compatibility across different SQL dialects.

## References
- [PostgreSQL Manual: SQL Syntax - Identifiers](https://www.postgresql.org/docs/current/sql-syntax-lexical.html#SQL-SYNTAX-IDENTIFIERS)
- [GORM Documentation: Raw SQL and SQL Builder](https://gorm.io/docs/sql_builder.html)
