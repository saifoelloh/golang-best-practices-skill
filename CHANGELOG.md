# Changelog

All notable changes to golang-fullstack-best-practices skill.

## [3.0.0] - 2026-03-13

### 🎉 Major Release: The Ultimate Fullstack Merger

Unified the general Go language best practices (v2.1.0) with the specialized GORM & PostgreSQL skill set (v1.0.0).

### Added

- **9 Specialized Domain Skills**:
  - `concurrency-safety` (12 rules)
  - `clean-architecture` (9 rules)
  - `design-patterns` (13 rules)
  - `idiomatic-go` (7 rules)
  - `error-handling` (**Unified**: 14 rules covering both Go and DB)
  - `gorm-query-patterns` (11 rules)
  - `postgresql-syntax` (10 rules)
  - `query-performance` (7 rules)
  - `migration-safety` (6 rules)
- **Unified Meta-Skill**: Root `SKILL.md` and `metadata.json` now coordinate **89 total rules**.

### Changed

- **Rebranded**: `golang-best-practices` → `golang-fullstack-best-practices`.
- **Merged Error Handling**: Consolidated general Go error patterns (wrapping, shadowing, leaks) with DB-specific patterns (PgError codes, transaction retries).
- **Metadata Update**: Version bumped to `3.0.0`.
- **Cleanup**: Removed redundant `database-repository` and old planning documents.

---

## [2.1.0] - 2026-03-13

### 🎉 Feature: Database Skill Standardization & Project Update

Standardized the recently added `database-repository` skill and performed a project-wide versioning sync.

### Added

- **Standardized Database & Repository Skill**:
  - Restructured `database-repository` to use individual rule files in `rules/` directory.
  
### Changed

- **Project Versioning**: Unified versioning across all skills to **v2.1.0**.
- **Rule Count**: Updated total rule count from **48** to **50**.

---

## [2.0.0] - 2026-01-22

### 🎉 Major Release: Multi-Skill Architecture

**Breaking Change**: Restructured from monolithic skill to 5 specialized domain skills.

### Added

**5 Domain-Specific Skills**: Concurrency, Architecture, Error Handling, Design Patterns, Idiomatic Go.
