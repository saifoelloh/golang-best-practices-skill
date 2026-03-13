# Golang Best Practices Skill

> Production-ready Go code review skill for AI agents based on authoritative sources

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Rules](https://img.shields.io/badge/Rules-50-blue.svg)](.)
[![Version](https://img.shields.io/badge/Version-2.1.0-green.svg)](./metadata.json)
[![Skills](https://img.shields.io/badge/Skills-6_domains-purple.svg)](#-available-skills)

**v2.1.0**: Now organized as **6 specialized domain skills** for faster, more focused code reviews.

## 📋 Overview

This skill contains **50 rules** across 6 specialized domains for comprehensive Go code review.

## 📚 Available Skills

| Skill | Rules | Use When | Trigger |
|-------|-------|----------|---------|
| **[Concurrency Safety](concurrency-safety/)** | 12 | Goroutines, channels, race conditions | "Check for race conditions" |
| **[Clean Architecture](clean-architecture/)** | 9 | Layer separation, dependency rules | "Audit architecture" |
| **[Error Handling](error-handling/)** | 7 | Error wrapping, context propagation | "Review error handling" |
| **[Design Patterns](design-patterns/)** | 13 | Refactoring, code smells | "Refactor this code" |
| **[Idiomatic Go](idiomatic-go/)** | 7 | Go-specific idioms | "Is this idiomatic Go?" |
| **[Database & Repository](database-repository/)** | 2 | GORM, PostgreSQL, SQL quoting | "Review repository layer" |

**Priority Distribution**:
- **8 CRITICAL** - Prevents production bugs, crashes, and failures
- **15 HIGH** - Improves reliability and architecture  
- **27 MEDIUM** - Enhances code quality and idioms
- **5 ARCHITECTURE** - Clean Architecture compliance

All rules are evidence-based from authoritative sources:
- "Learning Go: An Idiomatic Approach" by Jon Bodner
- "Concurrency in Go" by Katherine Cox-Buday  
- "Clean Architecture" by Robert C. Martin
- "Refactoring" by Martin Fowler
- "Design Patterns" by Gang of Four
- Refactoring.Guru

## 🚀 Quick Start

### Installation

Copy the `golang-best-practices-skill` directory to your skills location:

```bash
# Clone or copy to your skills directory
cp -r golang-best-practices-skill ~/.agent-skills/
```

### Usage

**Comprehensive audit** (meta-skill):
```
"Review my Go code for anti-patterns"
"Audit this Go service"
```

**Domain-specific reviews** (faster, focused):
```
"Check for race conditions"           → Concurrency Safety skill
"Audit architecture"                  → Clean Architecture skill
"Review error handling"               → Error Handling skill
"Refactor this complex function"      → Design Patterns skill  
"Is this idiomatic Go?"               → Idiomatic Go skill
"Review repository layer"             → Database & Repository skill
```

## 🎯 Why Multi-Skill Architecture?

**v1.x**: Single monolithic skill with 43 rules  
**v2.1**: 6 focused skills with 50 rules

### Benefits

- ⚡ **60-80% faster** agent processing for targeted reviews
- 🎯 **More accurate** results (focused context)
- 📚 **Better organization** (clear skill boundaries)
- 🔧 **Easier maintenance** (modify one skill at a time)
- 📈 **Scalable** (easy to add new skills)

## 🏗️ Structure

```
golang-best-practices-skill/
├── SKILL.md                    # Meta-skill coordinator
├── README.md                   # This file
├── metadata.json               # Skill metadata
│
├── concurrency-safety/         # 12 rules
│   ├── SKILL.md
│   └── rules/
│
├── clean-architecture/         # 9 rules
│   ├── SKILL.md
│   └── rules/
│
├── error-handling/             # 7 rules
│   ├── SKILL.md
│   └── rules/
│
├── design-patterns/            # 13 rules
│   ├── SKILL.md
│   └── rules/
│
├── idiomatic-go/               # 7 rules
│   ├── SKILL.md
│   └── rules/
│
├── database-repository/        # 2 rules
│   ├── SKILL.md
│   └── rules/                  # PostgreSQL & GORM rules
│
├── shared/                     # Common resources
│   ├── _categories.md
│   └── templates/
│
└── references/                 # Deep-dive guides
    ├── concurrency-deep-dive.md
    ├── error-handling-guide.md
    └── testing-strategies.md
```

## 📖 Skill Details

### 1. Concurrency Safety (12 rules)

Detects common concurrency bugs: goroutine leaks, race conditions, deadlocks, and unsafe channel operations.

**Rules**: 5 Critical + 4 High + 3 Medium

### 2. Clean Architecture (9 rules)

Ensures proper layering, dependency rules, and separation of concerns in gRPC → Usecase → Repository → Domain architectures.

**Rules**: 4 High + 5 Architecture

### 3. Error Handling (7 rules)

Ensures proper error wrapping, context propagation, and error checking.

**Rules**: 3 Critical + 3 High + 1 Medium

### 4. Design Patterns (13 rules)

Detects code smells and suggests pattern-based refactoring.

**Rules**: 2 High + 11 Medium

### 5. Idiomatic Go (7 rules)

Ensures code follows Go conventions and best practices.

**Rules**: 1 High + 6 Medium

### 6. Database & Repository (2 rules)

Focuses on safe GORM usage, PostgreSQL compatibility, and identifier quoting.

**Rules**: 1 High + 1 Medium

**Examples**:
- Reserved keyword quoting in raw SQL
- Redundant quoting in GORM Select
- PostgreSQL compatibility patterns

## ✨ Features

- **Evidence-Based**: All rules cite authoritative sources
- **Actionable**: Every rule includes before/after examples
- **Prioritized**: Focus on critical issues first
- **Production-Tested**: Validated on real codebases
- **Architecture-Aware**: Tailored for gRPC/Clean Architecture
- **Fast**: Domain-specific skills = faster agent processing

## 🎯 Use Cases

### Code Reviews
Automatically detect anti-patterns during PR reviews using domain-specific skills

### Refactoring
Identify technical debt and improvement opportunities with Design Patterns skill

### Onboarding
Help junior developers learn Go best practices through focused skills

### Architecture Audits
Ensure Clean Architecture compliance with dedicated architecture skill

### CI/CD Integration
Gate deployments on critical rule compliance

## 🤝 Contributing

Contributions welcome! To add a rule:

1. Choose appropriate skill domain
2. Use `shared/templates/_template.md` as starting point
3. Include before/after code examples
4. Cite authoritative source
5. Add to skill's SKILL.md
6. Update this README if needed

## 📚 References

### Books
- [Learning Go](https://www.oreilly.com/library/view/learning-go/9781492077206/) by Jon Bodner
- [Concurrency in Go](https://www.oreilly.com/library/view/concurrency-in-go/9781491941294/) by Katherine Cox-Buday
- [Clean Architecture](https://www.oreilly.com/library/view/clean-architecture-a/9780134494272/) by Robert C. Martin
- [Refactoring](https://martinfowler.com/books/refactoring.html) by Martin Fowler

### Online Resources
- [Effective Go](https://go.dev/doc/effective_go)
- [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments)
- [Uber Go Style Guide](https://github.com/uber-go/guide/blob/master/style.md)
- [Refactoring.Guru](https://refactoring.guru/)

## 📄 License

MIT License - see [LICENSE](./LICENSE) file for details

## 🙏 Acknowledgments

- Jon Bodner for "Learning Go"
- Katherine Cox-Buday for "Concurrency in Go"
- Robert C. Martin for "Clean Architecture"
- Martin Fowler for "Refactoring"
- The Go team for excellent documentation

## 👤 Author

**saifoelloh**
- Email: saifoelloh@gmail.com
- GitHub: [@saifoelloh](https://github.com/saifoelloh)
- Repository: [golang-best-practices-skill](https://github.com/saifoelloh/golang-best-practices-skill)

## 📞 Support

For issues or questions, please open an issue in the repository.

---

**Built with ❤️ for better Go code**
