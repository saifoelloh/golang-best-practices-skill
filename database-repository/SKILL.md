---
name: golang-database-repository
description: Database and Repository patterns for Go services using GORM and PostgreSQL. Focuses on safe identifier quoting, transaction handling, and SQL performance.
license: MIT
metadata:
  author: saifoelloh
  version: "2.1.0"
  parent_skill: golang-best-practices
  sources:
    - "GORM Documentation"
    - "PostgreSQL Manual: SQL Syntax"
  last_updated: "2026-03-13"
---

# Golang Database & Repository Patterns

Expert-level review for Database access and Repository patterns. Focuses on GORM-specific behaviors, PostgreSQL compatibility, and data layer safety.

## When to Apply

Use this skill when:
- Reviewing Repository layer code
- Debugging SQL errors related to reserved keywords (e.g., `order`, `group`)
- Ensuring PostgreSQL compatibility in a GORM-based project
- Auditing raw SQL fragments vs GORM-managed queries

## Rules Covered (2 total)

### Critical & High Impact

- `high-gorm-identifier-quoting` - Quote reserved keywords in raw SQL fragments
- `medium-gorm-select-redundancy` - Avoid manual quotes in simple `.Select()` calls

## Detailed Rules

Check each rule's detailed documentation in the `rules/` directory:

1. [GORM Identifier Quoting](rules/high-gorm-identifier-quoting.md) (`high-gorm-identifier-quoting`)
2. [GORM Select Redundancy](rules/medium-gorm-select-redundancy.md) (`medium-gorm-select-redundancy`)

## Trigger Phrases

This skill activates when you say:
- "Review repository layer"
- "Check SQL quoting"
- "Verify GORM PostgreSQL compatibility"
- "Check for reserved keyword issues in SQL"

## Philosophy

- **Explicit where necessary**: Quote reserved words in raw SQL fragments.
- **Let the library work**: Trust GORM's automatic quoting for simple fields.
- **Portability**: Use double quotes (`"`) as the SQL standard for identifiers (PostgreSQL/Oracle/SQLite compatible).
