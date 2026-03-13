---
title: Use CTEs for Complex Multi-Step Queries Instead of Nested Subqueries
impact: MEDIUM
impactDescription: Nested subqueries are unreadable and hard to debug — CTEs are equivalent but clear
tags: postgresql, cte, with-query, subquery, readability, debugging
source: PostgreSQL Docs - WITH Queries (CTEs)
---

## Use CTEs for Complex Multi-Step Queries Instead of Nested Subqueries

Common Table Expressions (CTEs, `WITH` clauses) decompose complex queries into named, readable steps. They are semantically equivalent to nested subqueries in most cases but dramatically easier to read, debug, and maintain.

**Impact**: MEDIUM - Readability, debuggability, maintainability of complex queries

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - nested subquery, hard to isolate and test each part
db.Raw(`
    SELECT * FROM orders WHERE user_id IN (
        SELECT id FROM users WHERE created_at > NOW() - INTERVAL '30 days'
        AND id IN (SELECT user_id FROM subscriptions WHERE status = 'active')
    )
`).Scan(&orders)
```

### Correct (Best Practice)

```go
// ✅ GOOD - CTE: each step is named and readable
db.Raw(`
    WITH recent_users AS (
        SELECT id FROM users
        WHERE created_at > NOW() - INTERVAL '30 days'
    ),
    active_subscribers AS (
        SELECT user_id FROM subscriptions WHERE status = 'active'
    )
    SELECT o.* FROM orders o
    JOIN recent_users ru ON ru.id = o.user_id
    JOIN active_subscribers a ON a.user_id = o.user_id
`).Scan(&orders)
```

### References

- [PostgreSQL Docs: WITH Queries](https://www.postgresql.org/docs/16/queries-with.html)

---

**Rule Version**: 1.0
**Last Updated**: 2026-03-13
**Priority**: MEDIUM
**Auto-fixable**: No
