---
name: gorm-sql-postgresql-syntax
description: PostgreSQL SQL syntax review for Go/GORM projects. Use when writing or reviewing raw SQL used via db.Raw(), db.Exec(), or migrations. Covers placeholder style, reserved keyword quoting, RETURNING, ILIKE, JSONB operators, and type casting. Don't use for MySQL or SQLite.
license: MIT
metadata:
  author: saifoelloh
  version: "1.0.0"
  parent_skill: gorm-sql-best-practices
  sources:
    - "PostgreSQL 16 Official Documentation"
    - "The Art of PostgreSQL (Dimitri Fontaine)"
    - "GORM Official Documentation (gorm.io/docs)"
  last_updated: "2026-03-13"
---

# PostgreSQL Syntax

Expert-level review of raw PostgreSQL SQL syntax in GORM/Go projects. Ensures correct Postgres-specific usage, idiomatic SQL, and avoids common runtime errors.

## When to Apply

Use this skill when:
- Writing raw SQL in `db.Raw()`, `db.Exec()`, or migration files
- Debugging `syntax error` or `operator does not exist` PostgreSQL errors
- Reviewing SQL expressions involving JSONB, UUID, timestamps, or reserved keywords
- Using `RETURNING`, CTEs, window functions, or `ILIKE`

## Rule Categories

| Priority | Count | Focus |
|---|---|---|
| Critical | 1 | Placeholder style mismatch |
| High | 4 | Identifier quoting, RETURNING, ILIKE, type casting |
| Medium | 4 | CTE readability, JSONB operators, INTERVAL, select redundancy |

## Rules Covered (9 total)

### Critical Issues (1)
- `critical-placeholder-style` — Use `?` with GORM, `$N` only with pgx/pq directly

### High-Impact Patterns (4)
- `high-identifier-quoting` — Quote reserved keywords (`order`, `user`, `group`) in raw SQL
- `high-returning-clause` — Use `RETURNING` for generated values after INSERT/UPDATE
- `high-ilike-search` — Use `ILIKE` for case-insensitive search, not `LOWER(col) LIKE`
- `high-explicit-cast` — Cast types explicitly (`::uuid`, `::bigint`) to avoid `operator does not exist`

### Medium Improvements (4)
- `medium-cte-readability` — Use CTEs instead of nested subqueries for complex queries
- `medium-jsonb-operators` — Use correct `@>`, `->>`, `->` operators for JSONB queries
- `medium-interval-arithmetic` — Use `INTERVAL` for date math, not manual second calculations
- `medium-select-redundancy` — Don't manually quote plain column names in `.Select()` (GORM handles it)

## Trigger Phrases

- "Check this raw SQL"
- "db.Raw / db.Exec"
- "SQL syntax error"
- "operator does not exist"
- "reserved keyword"
- "JSONB query"
- "PostgreSQL compatibility"

## Cross-Reference with golang-best-practices-skill

Two rules in this skill directly extend rules from the companion `database-repository` skill in [golang-best-practices-skill](https://github.com/saifoelloh/golang-best-practices-skill):

| This skill | golang-best-practices-skill | Relationship |
|---|---|---|
| `high-identifier-quoting` | `high-gorm-identifier-quoting` | Expanded — more Postgres examples, full reserved word list |
| `medium-select-redundancy` | `medium-gorm-select-redundancy` | Mirror — same rule, cross-referenced |

## Related Skills

- [gorm-query-patterns](../gorm-query-patterns/SKILL.md) — For ORM-level query patterns
- [query-performance](../query-performance/SKILL.md) — For index-related syntax (CONCURRENTLY, partial indexes)
- [migration-safety](../migration-safety/SKILL.md) — For DDL SQL in migration files
