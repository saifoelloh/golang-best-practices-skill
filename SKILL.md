---
name: golang-fullstack-best-practices
description: Comprehensive Go + GORM + PostgreSQL review meta-skill. Coordinates 9 specialized domain skills for fullstack Go application audits. Covers concurrency, architecture, idioms, queries, SQL performance, and migration safety.
license: MIT
metadata:
  author: saifoelloh
  version: "3.0.0"
  skill_type: meta
  sub_skills:
    - concurrency-safety
    - clean-architecture
    - design-patterns
    - idiomatic-go
    - error-handling
    - gorm-query-patterns
    - postgresql-syntax
    - query-performance
    - migration-safety
  last_updated: "2026-03-13"
---

# Golang Fullstack Best Practices (Meta-Skill)

Comprehensive code review skill coordinating **9 specialized domain skills** for Go backend services using GORM with PostgreSQL.

> **v3.0.0**: The "Ultimate Merger" release. Unified 89 rules across Go language and Database layers.

## Available Skills

### Go Language Domains
1. **[Concurrency Safety](concurrency-safety/SKILL.md)** — Goroutines, channels, race conditions.
2. **[Clean Architecture](clean-architecture/SKILL.md)** — Layered architecture, gRPC, dependency rules.
3. **[Design Patterns](design-patterns/SKILL.md)** — Code smells, GoF patterns, refactoring.
4. **[Idiomatic Go](idiomatic-go/SKILL.md)** — Go conventions, interface design, pointers.

### Database Layer Domains
5. **[GORM Query Patterns](gorm-query-patterns/SKILL.md)** — ORM queries, transactions, context.
6. **[PostgreSQL Syntax](postgresql-syntax/SKILL.md)** — Raw SQL, Postgres-specific functions, quoting.
7. **[Query Performance](query-performance/SKILL.md)** — N+1 detection, indexing, connection pooling.
8. **[Migration Safety](migration-safety/SKILL.md)** — Zero-downtime schema changes, rollbacks.

### Unified Core
9. **[Error Handling](error-handling/SKILL.md)** — Combined Go error patterns + PostgreSQL error codes.

## Rule Count by Priority

| Priority | Count | Focus |
|---|---|---|
| **CRITICAL** | 20 | Bugs, crashes, data loss, SQL injection |
| **HIGH** | 30 | Reliability, performance, architecture |
| **MEDIUM** | 34 | Quality, idioms, maintainability |
| **ARCH** | 5 | Clean Architecture compliance |
| **TOTAL** | **89** | |

## Unified Routing Logic

```
- Concurrency/Channels/Goroutines → concurrency-safety
- Layers/gRPC/Usecases/Dipendencies → clean-architecture
- Complex logic/Refactoring/Code smells → design-patterns
- Pointer usage/Interfaces/Idioms → idiomatic-go
- DB Errors/Postgres codes/errors.Is/As → error-handling
- GORM queries/db.Find/Transactions → gorm-query-patterns
- Raw SQL/Pg-specific functions/Syntax → postgresql-syntax
- Slow queries/Indexes/N+1/Pool → query-performance
- ALTER TABLE/AutoMigrate/Rollback → migration-safety
- Full Audit → all 9 skills
```

## How to Use

### For Full Audit
"Review this Go service for all language and database best practices"

### For Targeted Reviews
"Audit this repository file for GORM query performance and N+1 issues"
"Review these goroutines for race conditions and context leaks"
"Is this database migration safe for production?"
