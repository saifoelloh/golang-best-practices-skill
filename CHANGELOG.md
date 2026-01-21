# Changelog

All notable changes to golang-best-practices skill.

## [1.1.0] - 2026-01-22

### Added

**Design Patterns & Refactoring Rules (13 new rules)**

HIGH Priority:
- `high-god-object` - Extract logic from 300+ line functions
- `high-extract-method` - Name complex code blocks with descriptive methods

MEDIUM Priority:
- `medium-primitive-obsession` - Replace primitives with value objects
- `medium-long-parameter-list` - Use parameter objects for >5 params
- `medium-data-clumps` - Extract repeated parameter groups
- `medium-feature-envy` - Move logic closer to data
- `medium-magic-constants` - Replace magic numbers with named constants
- `medium-builder-pattern` - Fluent API for complex construction
- `medium-factory-constructor` - Validated object creation
- `medium-introduce-parameter-object` - Group related parameters
- `medium-switch-to-strategy` - Replace type switches with interfaces
- `medium-middleware-decorator` - Decorator pattern for http.Handler
- `medium-law-of-demeter` - Reduce coupling, avoid message chains

### Changed

- Updated version from 1.0.0 to 1.1.0
- Rule count: 30 → 43 (+43% coverage)
- Enhanced documentation with refactoring.guru references
- Added real-world Go examples from production code

### References

- Added citations to Refactoring.Guru
- Integrated Martin Fowler's Refactoring catalog
- References to Gang of Four Design Patterns
- Clean Code principles from Robert C. Martin

## [1.0.0] - 2026-01-21

### Added

- Initial release with 30 core rules
- 8 CRITICAL rules for bug prevention
- 12 HIGH priority rules for correctness
- 10 MEDIUM rules for code quality
- Coverage: error handling, concurrency, Clean Architecture
- Examples from production Go code
- Comprehensive references to Go documentation

---

**Versioning:** This project follows [Semantic Versioning](https://semver.org/)
