---
title: Rule Title Here
impact: CRITICAL | HIGH | MEDIUM | ARCHITECTURE
impactDescription: Brief description of impact (e.g., "Memory leaks", "20-50% performance improvement")
tags: tag1, tag2, tag3
source: Book Name - Chapter X
---

## Rule Title Here

Brief explanation of the rule and why it matters. This should be clear and concise, explaining the implications of violating this rule.

**Impact**: [CRITICAL/HIGH/MEDIUM] - [What happens if violated]

### Detection

How to spot this issue in code:
- Specific pattern to look for
- Code smell indicators
- AST patterns (if applicable)

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Explanation of what's wrong
func badExample() {
    // Code demonstrating the anti-pattern
    // with inline comments explaining the problem
}
```

**Why this is wrong**:
- Reason 1
- Reason 2

### Correct (Best Practice)

```go
// ✅ GOOD - Explanation of what's right
func goodExample() {
    // Code demonstrating the correct pattern
    // with inline comments explaining the solution
}
```

**Why this is better**:
- Benefit 1
- Benefit 2

### Additional Context

Any extra information, edge cases, or related patterns:
- When to apply this rule
- Exceptions (if any)
- Related rules to consider
- Performance implications

### References

- [Go Documentation](https://go.dev/doc/)
- [Effective Go](https://go.dev/doc/effective_go)
- Specific book chapter or blog post

---

**Rule Template Version**: 1.0  
**Last Updated**: 2026-01-21
