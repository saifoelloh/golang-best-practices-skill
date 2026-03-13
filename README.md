# Golang Fullstack Best Practices Skill

> Comprehensive Go + GORM + PostgreSQL review skill for AI agents based on authoritative sources

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Rules](https://img.shields.io/badge/Rules-89-blue.svg)](.)
[![Version](https://img.shields.io/badge/Version-3.0.0-green.svg)](./metadata.json)
[![Domains](https://img.shields.io/badge/Domains-9-purple.svg)](#-available-skills)

**v3.0.0**: The "Ultimate Merger" release. Unified **89 rules** across **9 specialized domains**, covering everything from Go concurrency and architecture to deep PostgreSQL performance and migration safety.

## 📋 Overview

This meta-skill provides a complete code review solution for Go fullstack backend services. It combines general Go language best practices (v2.1.0) with specialized GORM & PostgreSQL expertise (v1.0.0).

## 📚 Available Skills

| Domain | Rules | Focus |
|---|---|---|
| **[Concurrency Safety](concurrency-safety/)** | 12 | Goroutines, channels, race conditions |
| **[Clean Architecture](clean-architecture/)** | 9 | Layered architecture, gRPC, dependency inversion |
| **[Error Handling (Unified)](error-handling/)** | 14 | Go error chains + Postgres error codes/retries |
| **[Design Patterns](design-patterns/)** | 13 | Code smells, refactoring, GoF patterns |
| **[Idiomatic Go](idiomatic-go/)** | 7 | Go conventions, interface design, pointers |
| **[GORM Query Patterns](gorm-query-patterns/)** | 11 | ORM queries, transactions, context |
| **[PostgreSQL Syntax](postgresql-syntax/)** | 10 | Raw SQL, Postgres-specific features |
| **[Query Performance](query-performance/)** | 7 | N+1 detection, indexing, EXPLAIN |
| **[Migration Safety](migration-safety/)** | 6 | Zero-downtime schema changes, rollbacks |

**Priority Distribution**:
- **20 CRITICAL** — Data loss, SQL injection, crashes, production bugs
- **30 HIGH** — Reliability, performance, architecture
- **34 MEDIUM** — Code quality, idiomatic patterns
- **5 ARCHITECTURE** — Structural compliance

## 🚀 Quick Start

### Usage

**Full Audit**:
```
"Review this Go project for all language and database anti-patterns"
```

**Targeted Reviews**:
```
"Check for race conditions"              → Concurrency Safety
"Audit architecture"                     → Clean Architecture
"Review GORM queries for N+1"            → GORM Patterns / Query Performance
"Debug this 23505 Postgres error"        → Error Handling
"Is this migration safe for production?" → Migration Safety
```

## 🏗️ Structure

```
golang-fullstack-best-practices/
├── SKILL.md                    # Meta-skill coordinator (v3.0.0)
├── metadata.json               # Full rule manifest
│
├── concurrency-safety/         # Go-specific
├── clean-architecture/         # Architecture
├── design-patterns/            # Quality & Smells
├── idiomatic-go/               # Conventions
│
├── gorm-query-patterns/        # GORM ORM
├── postgresql-syntax/          # Raw SQL
├── query-performance/          # Optimization
├── migration-safety/           # DB Schema changes
│
├── error-handling/             # Unified Go & DB
├── shared/                     # Rule templates & categories
└── references/                 # Deep-dive guides
```

## ✨ Features

- **End-to-End Coverage**: From Go syntax to PostgreSQL index optimization.
- **Unified Routing**: High-accuracy targeting using meta-skill coordination.
- **Evidence-Based**: Rules cite 8+ authoritative sources (Jon Bodner, Martin Fowler, Robert Martin, PostgreSQL Docs, etc.).
- **Production-Ready**: Focused on zero-downtime, safe operations, and debugging real-world failures.

## 👤 Author

**saifoelloh**
- GitHub: [@saifoelloh](https://github.com/saifoelloh)
- Repository: [golang-best-practices-skill](https://github.com/saifoelloh/golang-best-practices-skill)

---

**Built with ❤️ for bulletproof Go systems.**
