---
title: Quote Reserved Keywords in Raw SQL Fragments
impact: HIGH
impactDescription: Reserved keywords like order, user, group unquoted in SQL cause runtime syntax errors
tags: postgresql, gorm, identifier-quoting, reserved-keywords, sql-syntax, order, user, group
source: PostgreSQL Manual - SQL Syntax Identifiers, GORM Docs - Raw SQL and SQL Builder
---

## Quote Reserved Keywords in Raw SQL Fragments

GORM does not automatically quote identifiers in raw SQL fragments like `.Where()`, `.Order()`, or `gorm.Expr()`. Column or table names that are PostgreSQL reserved keywords (`order`, `user`, `group`, `limit`, `offset`, `check`, `default`) will cause syntax errors if unquoted.

**Impact**: HIGH - `ERROR: syntax error at or near "order"` at runtime for any query using reserved-keyword column names

### Detection

How to spot this issue:
- Reserved words (`order`, `user`, `group`, `limit`, `offset`) in string arguments of `.Order()`, `.Where()`, or inside `gorm.Expr()`
- Column named `order`, `user`, `group`, etc. in model struct but used raw in SQL fragments
- `db.Select("COALESCE(MAX(order), 0)")` — reserved word in SQL expression without quotes

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - "order" is a reserved keyword → syntax error
db.Order("order ASC").Find(&results)
// ERROR: syntax error at or near "order"

// ❌ BAD - unquoted in WHERE
db.Where("order = ?", 1).First(&result)
// ERROR: syntax error at or near "="

// ❌ BAD - unquoted in expression
db.Select("COALESCE(MAX(order), 0)")
// ERROR: syntax error at or near ")"

// ❌ BAD - "user" table without quoting
db.Raw("SELECT * FROM user WHERE id = ?", id)
// ERROR: syntax error at or near "WHERE"
```

**Why this is wrong**:
- PostgreSQL's parser sees `order`, `user`, `group` as SQL keywords, not identifiers
- GORM only auto-quotes in its ORM layer (struct field mapping) — raw string fragments bypass this

### Correct (Best Practice)

```go
// ✅ GOOD - double-quote reserved keywords (PostgreSQL standard)
db.Order(`"order" ASC`).Find(&results)
db.Where(`"order" = ?`, 1).First(&result)
db.Select(`COALESCE(MAX("order"), 0)`)

// ✅ GOOD - backtick raw string literal for readability
db.Raw(`SELECT id, "order", status FROM shipments WHERE "order" = ?`, orderNum).
    Scan(&results)

// ✅ GOOD - avoid the problem by renaming the column (best long-term fix)
// Prefer: order_number, order_seq, sort_order instead of "order"
type Shipment struct {
    OrderNumber int `gorm:"column:order_number"`  // No quoting needed
}

// ✅ GOOD - GORM auto-quotes struct field names in ORM methods
db.Where(&Shipment{OrderNumber: 1}).First(&result)  // Safe, GORM handles quoting
```

**Why this is better**:
- Double-quoted identifiers are the SQL standard (ANSI SQL) — works on PostgreSQL, Oracle, SQLite
- Raw string literals (backtick) avoid escaping hell with `\"`
- Renaming columns to non-reserved words is the cleanest long-term solution

### Full List of Common Reserved Keywords to Watch

`order`, `user`, `group`, `limit`, `offset`, `check`, `default`, `table`, `index`, `key`, `value`, `type`, `name`, `date`, `time`, `timestamp`, `end`, `begin`

### Additional Context

> **Note from golang-best-practices-skill**: This rule was introduced in the `database-repository` domain of the companion Go skill (`high-gorm-identifier-quoting`). The SQL skill expands on it with PostgreSQL-specific examples and the full quoting reference. See also `medium-select-redundancy.md`.

### References

- [PostgreSQL Manual: SQL Syntax — Identifiers](https://www.postgresql.org/docs/16/sql-syntax-lexical.html#SQL-SYNTAX-IDENTIFIERS)
- [GORM Documentation: Raw SQL and SQL Builder](https://gorm.io/docs/sql_builder.html)
- [PostgreSQL Reserved Words List](https://www.postgresql.org/docs/16/sql-keywords-appendix.html)

---

**Rule Version**: 1.0
**Last Updated**: 2026-03-13
**Priority**: HIGH
**Auto-fixable**: Partially (known reserved words can be auto-quoted)
