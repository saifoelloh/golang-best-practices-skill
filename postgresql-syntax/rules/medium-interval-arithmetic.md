---
title: Use INTERVAL for Date Arithmetic, Not Manual Second Calculations
impact: MEDIUM
impactDescription: Manual second math is error-prone and unreadable — INTERVAL is expressive and correct
tags: postgresql, interval, date-arithmetic, timestamp, readability
source: PostgreSQL Docs - Date/Time Functions
---

## Use INTERVAL for Date Arithmetic, Not Manual Second Calculations

Calculating time deltas by multiplying seconds is error-prone (leap years, DST) and unreadable. PostgreSQL's `INTERVAL` type handles calendar arithmetic correctly and is self-documenting.

**Impact**: MEDIUM - Bug risk from manual time math, reduced readability

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - magic number, doesn't account for DST, hard to read
db.Raw("SELECT * FROM sessions WHERE created_at > NOW() - ?", 30*24*60*60)
// What is 2592000? Not obvious. Also: integer seconds may cause type errors.
```

### Correct (Best Practice)

```go
// ✅ GOOD - readable and correct
db.Raw("SELECT * FROM sessions WHERE created_at > NOW() - INTERVAL '30 days'").Scan(&sessions)

// ✅ GOOD - parameterized interval
db.Raw("SELECT * FROM sessions WHERE created_at > NOW() - (? || ' days')::INTERVAL", 30).Scan(&sessions)

// ✅ GOOD - in Go, use time.Duration for the boundary and pass as time.Time
cutoff := time.Now().AddDate(0, 0, -30)
db.Where("created_at > ?", cutoff).Find(&sessions)
```

### References

- [PostgreSQL Docs: Date/Time Functions](https://www.postgresql.org/docs/16/functions-datetime.html)

---

**Rule Version**: 1.0
**Last Updated**: 2026-03-13
**Priority**: MEDIUM
**Auto-fixable**: No
