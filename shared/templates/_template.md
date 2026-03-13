---
title: Rule Title Here
impact: CRITICAL | HIGH | MEDIUM
impactDescription: Brief description of impact (e.g., "SQL injection", "Full table scan on JOIN")
tags: gorm, postgresql, tag1, tag2
source: Source Name - Section/Chapter (e.g., "GORM Docs - Error Handling")
---

## Rule Title Here

Brief explanation of the rule and why it matters. Should be clear and concise, explaining the implications of violating this rule in a GORM + PostgreSQL context.

**Impact**: [CRITICAL/HIGH/MEDIUM] - [What happens if violated]

### Detection

How to spot this issue in code:
- Specific GORM method or SQL pattern to look for
- Code smell indicators
- Common anti-pattern shape

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Explanation of what's wrong
func (r *SomeRepository) BadMethod(ctx context.Context) error {
    // Code demonstrating the anti-pattern
    // with inline comments explaining the problem
    return nil
}
```

**Why this is wrong**:
- Reason 1
- Reason 2

### Correct (Best Practice)

```go
// ✅ GOOD - Explanation of what's right
func (r *SomeRepository) GoodMethod(ctx context.Context) error {
    // Code demonstrating the correct pattern
    // with inline comments explaining the solution
    return nil
}
```

**Why this is better**:
- Benefit 1
- Benefit 2

### Additional Context

Any extra information, edge cases, or related patterns:
- When this rule applies vs exceptions
- Related rules in this skill or in golang-best-practices-skill
- Performance implications
- PostgreSQL-specific notes

### References

- [GORM Docs: Relevant Section](https://gorm.io/docs/)
- [PostgreSQL Docs: Relevant Section](https://www.postgresql.org/docs/16/)
- Book or blog reference if applicable

---

**Rule Version**: 1.0
**Last Updated**: 2026-03-13
**Priority**: CRITICAL | HIGH | MEDIUM
**Auto-fixable**: Yes | No | Partially
