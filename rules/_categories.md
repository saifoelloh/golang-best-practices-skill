# Rule Categories Index

This document lists all rules organized by category and priority.

---

## Critical Issues (Priority 1) - 8 Rules

**Impact**: Prevents bugs, crashes, memory leaks, and production failures

| Rule | File | Impact |
|------|------|--------|
| Error Wrapping | `critical-error-wrapping.md` | Use %w to preserve error chain |
| Defer in Loop | `critical-defer-in-loop.md` | Resource leaks from deferred closures |
| Context Leak | `critical-context-leak.md` | Always defer cancel() after context creation |
| Error Shadow | `critical-error-shadow.md` | Don't shadow err variable in nested scopes |
| Goroutine Leak | `critical-goroutine-leak.md` | Goroutines must have exit conditions |
| Race Condition | `critical-race-condition.md` | Protect shared state with mutex/channels |
| Channel Deadlock | `critical-channel-deadlock.md` | Ensure paired send/receive operations |
| Close Panic | `critical-close-panic.md` | Sender closes channel, not receiver |

---

## High-Impact Patterns (Priority 2) - 12 Rules

**Impact**: Reliability, correctness, and performance improvements

| Rule | File | Impact |
|------|------|--------|
| Pointer Receiver | `high-pointer-receiver.md` | Use pointer receivers for mutations |
| Context Propagation | `high-context-propagation.md` | Propagate context through call chain |
| Error Is/As | `high-error-is-as.md` | Use errors.Is/As instead of == |
| Interface Nil | `high-interface-nil.md` | Check interface nil correctly |
| Goroutine Unbounded | `high-goroutine-unbounded.md` | Limit concurrent goroutines (worker pool) |
| Channel Not Closed | `high-channel-not-closed.md` | Always close channels when done |
| Loop Variable Capture | `high-loop-variable-capture.md` | Avoid closure over loop variables |
| WaitGroup Mismatch | `high-waitgroup-mismatch.md` | Match Add() and Done() calls |
| Business Logic Handler | `high-business-logic-handler.md` | Keep delivery layer thin |
| Business Logic Repository | `high-business-logic-repository.md` | No business logic in data layer |
| Constructor Creates Deps | `high-constructor-creates-deps.md` | Inject dependencies, don't create |
| Transaction in Repository | `high-transaction-in-repository.md` | Transactions belong in usecase |

---

## Medium Improvements (Priority 3) - 10 Rules

**Impact**: Code quality, idioms, and maintainability

| Rule | File | Impact |
|------|------|--------|
| Interface Pollution | `medium-interface-pollution.md` | Keep interfaces small (<5 methods) |
| Accept Interface Return Struct | `medium-accept-interface-return-struct.md` | API flexibility pattern |
| Pointer Overuse | `medium-pointer-overuse.md` | Don't overuse pointers for small types |
| Directional Channels | `medium-directional-channels.md` | Use send/receive-only channels |
| Buffered Channel Size | `medium-buffered-channel-size.md` | Choose appropriate buffer size |
| Select Default | `medium-select-default.md` | Avoid busy-wait with select |
| Fat Interface | `medium-fat-interface.md` | Split large interfaces |
| Usecase Complexity | `medium-usecase-complexity.md` | Move business logic to domain entities |
| Interface in Implementation | `medium-interface-in-implementation.md` | Define interfaces where used |
| Sentinel Error Usage | `medium-sentinel-error-usage.md` | Use sentinel errors for stable categories |

---

## Architecture (Priority 4) - 5 Rules

**Impact**: Clean Architecture compliance for Go services

| Rule | File | Impact |
|------|------|--------|
| Domain Import Infra | `arch-domain-import-infra.md` | Domain must not import infrastructure |
| Concrete Dependency | `arch-concrete-dependency.md` | Depend on interfaces, not concrete types |
| Repository Business Logic | `arch-repository-business-logic.md` | Repositories do CRUD only |
| Usecase Orchestration | `arch-usecase-orchestration.md` | Usecases orchestrate, entities decide |
| Interface Segregation | `arch-interface-segregation.md` | Small, consumer-defined interfaces |

---

## Total: 35 Rules

- **CRITICAL**: 8 rules (23%)
- **HIGH**: 12 rules (34%)
- **MEDIUM**: 10 rules (29%)
- **ARCHITECTURE**: 5 rules (14%)

---

## How to Navigate

Rules are stored in files matching their name:
```
rules/critical-error-wrapping.md
rules/high-pointer-receiver.md
rules/medium-interface-pollution.md
rules/arch-domain-import-infra.md
```

Each rule file follows the template in `_template.md` with:
- YAML frontmatter (title, impact, tags, source)
- Rule explanation
- Detection criteria
- Incorrect example (❌ BAD)
- Correct example (✅ GOOD)
- Additional context
- References

---

**Last Updated**: 2026-01-21
