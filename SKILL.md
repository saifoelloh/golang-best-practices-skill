---
name: golang-best-practices
description: Go code review and optimization skill. Use when asked to "review Go code", "check idiomatic Go", "optimize Golang", "find Go anti-patterns", or "improve Go performance". Covers error handling, concurrency, architecture, performance, and idiomatic patterns based on authoritative sources.
license: MIT
metadata:
  author: ekuid
  version: "1.1.0"
  sources:
    - "Learning Go: An Idiomatic Approach (Jon Bodner)"
    - "Concurrency in Go (Katherine Cox-Buday)"
    - "Clean Architecture (Robert C. Martin)"
    - "Refactoring (Martin Fowler)"
    - "Design Patterns (Gang of Four)"
    - "Refactoring.Guru"
  last_updated: "2026-01-22"
---

# Golang Best Practices

Comprehensive Go code review skill based on authoritative sources. Contains 43 rules across 8 categories, prioritized by impact to guide code reviews, refactoring, and optimization.

## When to Apply

Use this skill when:
- Reviewing Go code for bugs or anti-patterns
- Optimizing Go application performance
- Refactoring Go codebase for idiomatic patterns
- Auditing concurrency safety
- Implementing Clean Architecture in Go services
- Preparing Go code for production deployment

## Rule Categories by Priority

| Priority | Category | Impact | Prefix | Rule Count |
|----------|----------|--------|--------|------------|
| 1 | Critical Issues | CRITICAL | `critical-` | 8 |
| 2 | High-Impact Patterns | HIGH | `high-` | 14 |
| 3 | Medium Improvements | MEDIUM | `medium-` | 21 |
| 4 | Architecture | ARCHITECTURE | `arch-` | 5 |

## Quick Reference

### 1. Critical Issues (CRITICAL)

**These prevent bugs, crashes, and production failures**

- `critical-error-wrapping` - Use %w for error wrapping, not %v
- `critical-defer-in-loop` - Avoid defer in loops (resource leaks)
- `critical-context-leak` - Always defer cancel() after context creation
- `critical-error-shadow` - Don't shadow err variable in nested scopes
- `critical-goroutine-leak` - Goroutines must have exit conditions
- `critical-race-condition` - Protect shared state with mutex/channels
- `critical-channel-deadlock` - Ensure paired send/receive operations
- `critical-close-panic` - Sender closes channel, not receiver

### 2. High-Impact Patterns (HIGH)

**Reliability and correctness improvements**

- `high-pointer-receiver` - Use pointer receivers for mutations
- `high-context-propagation` - Propagate context through call chain
- `high-error-is-as` - Use errors.Is/As instead of ==
- `high-interface-nil` - Check interface nil correctly
- `high-goroutine-unbounded` - Limit concurrent goroutines (worker pool)
- `high-channel-not-closed` - Always close channels when done
- `high-loop-variable-capture` - Avoid closure over loop variables
- `high-waitgroup-mismatch` - Match Add() and Done() calls
- `high-business-logic-handler` - Keep delivery layer thin
- `high-business-logic-repository` - No business logic in data layer
- `high-constructor-creates-deps` - Inject dependencies, don't create
- `high-transaction-in-repository` - Transactions belong in usecase
- `high-god-object` - Extract logic from 300+ line functions
- `high-extract-method` - Name complex code blocks with descriptive methods

### 3. Medium Improvements (MEDIUM)

**Code quality and idioms**

- `medium-interface-pollution` - Keep interfaces small (<5 methods)
- `medium-accept-interface-return-struct` - API flexibility pattern
- `medium-pointer-overuse` - Don't overuse pointers for small types
- `medium-directional-channels` - Use send/receive-only channels
- `medium-buffered-channel-size` - Choose appropriate buffer size
- `medium-select-default` - Avoid busy-wait with select
- `medium-fat-interface` - Split large interfaces
- `medium-usecase-complexity` - Move business logic to domain entities
- `medium-interface-in-implementation` - Define interfaces where used
- `medium-sentinel-error-usage` - Use sentinel errors for stable categories
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

### 4. Architecture (ARCHITECTURE)

**Clean Architecture compliance for gRPC/usecase/repository pattern**

- `arch-domain-import-infra` - Domain must not import infrastructure
- `arch-concrete-dependency` - Depend on interfaces, not concrete types
- `arch-repository-business-logic` - Repositories do CRUD only
- `arch-usecase-orchestration` - Usecases orchestrate, entities decide
- `arch-interface-segregation` - Small, consumer-defined interfaces

## How to Use

### For Code Review

1. Read the code file(s)
2. Check against rules in priority order (Critical first)
3. For each violation found, reference the specific rule file
4. Provide exact line numbers and explanation
5. Show corrected code example

### For Refactoring

1. Identify code smells matching detection patterns
2. Apply fixes from corresponding rule files
3. Verify no regressions introduced
4. Run tests to confirm correctness

### Accessing Detailed Rules

Each rule file contains:
- Brief explanation of why it matters
- Detection criteria (how to spot the issue)
- Incorrect code example with explanation
- Correct code example with explanation
- Impact assessment
- Additional context and references

**Example**:
```
rules/critical-error-wrapping.md
rules/high-pointer-receiver.md
```

## Reference Guides

For deep dives on specific topics:

- `references/concurrency-deep-dive.md` - Comprehensive concurrency patterns
- `references/error-handling-guide.md` - Complete error handling strategies
- `references/testing-strategies.md` - Go testing best practices

## Common Usage Patterns

**Review entire file:**
```
Review this Go file for anti-patterns and suggest improvements
```

**Focus on specific category:**
```
Check this code for concurrency issues (CRITICAL level)
```

**Architecture audit:**
```
Verify this service follows Clean Architecture principles
```

**Performance optimization:**
```
Find performance issues in this Go code
```

## Trigger Phrases

This skill activates when you say:
- "Review my Go code"
- "Check this Golang file"
- "Find Go anti-patterns"
- "Optimize this Go function"
- "Is this idiomatic Go?"
- "Check for goroutine leaks"
- "Review error handling"
- "Audit Go architecture"
- "Check concurrency safety"

## Best Practices Philosophy

Based on research from authoritative sources:

**Jon Bodner's Principles**:
- Go is deliberately simple and explicit
- "Boring" code is good code - predictable and maintainable
- Understand the "why" behind language features
- Idiomatic Go prioritizes clarity over cleverness

**Katherine Cox-Buday's Guidelines**:
- Concurrency is not parallelism
- Channels are for communication, mutexes are for state
- Always have exit conditions for goroutines
- Context is the standard way to propagate cancellation

**Clean Architecture (Uncle Bob)**:
- Dependencies point inward toward business logic
- Domain layer has no external dependencies
- Interfaces defined by consumers, not producers
- Separation of concerns across layers

## Output Format

When reviewing code, use this format:

```
## Critical Issues Found: X

### [Rule Name] (Line Y)
**Issue**: Brief description
**Fix**: Suggested correction
**Example**:
```go
// Corrected code here
```

## Medium Improvements: X

[Similar format for medium priority items]
```

## Notes

- Rules are evidence-based from authoritative Go books
- Detection patterns help identify issues programmatically
- All rules include working code examples
- Tailored for gRPC/usecase/repository architecture patterns
