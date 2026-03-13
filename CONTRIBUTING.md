# Contributing to GORM + PostgreSQL Best Practices Skill

Contributions welcome! This skill is designed to help Go developers write correct, performant, and safe database code using GORM + PostgreSQL.

## How to Contribute

### Adding a New Rule

1. **Research** — Ensure the rule is backed by an authoritative source
   - Preferred: GORM official docs, PostgreSQL official docs, published books
   - Acceptable: Established engineering blog posts (Braintree, Uber, Cloudflare)
   - Avoid: Personal opinions without citations

2. **Choose the Right Domain**:
   - `gorm-query-patterns/` — GORM ORM API usage (db.Find, Preload, Updates, etc.)
   - `postgresql-syntax/` — Raw SQL syntax, Postgres-specific functions and types
   - `query-performance/` — Indexes, N+1, EXPLAIN, connection pool, batching
   - `error-handling/` — PostgreSQL error codes, errors.Is/As, retry logic
   - `migration-safety/` — DDL changes, rollback, idempotency, zero-downtime

3. **Use the Template** — Start with `shared/templates/_template.md`
   ```bash
   cp shared/templates/_template.md {domain}/rules/{priority}-{rule-name}.md
   ```
   Naming convention: `{critical|high|medium}-{kebab-case-rule-name}.md`

4. **Fill In All Required Sections**:
   - YAML frontmatter (`title`, `impact`, `impactDescription`, `tags`, `source`)
   - Rule explanation with real-world consequence
   - Detection criteria (what to look for in code)
   - Incorrect example (`❌ BAD`) with inline explanation
   - Correct example (`✅ GOOD`) with inline explanation
   - Additional context (edge cases, related rules)
   - References (linked, authoritative)

5. **Prioritize Correctly**:
   - **CRITICAL**: Production bugs, data loss, security vulnerabilities, crashes
   - **HIGH**: Performance degradation, subtle correctness bugs, broken error handling
   - **MEDIUM**: Code quality, idiomatic patterns, maintainability

6. **Update the Index**:
   - Add rule to `shared/_categories.md`
   - Add rule to the domain's `SKILL.md` rules table
   - Add rule to root `SKILL.md` quick reference table
   - Update `metadata.json` rule counts

### Testing Your Rule

Before submitting, verify:
1. The anti-pattern example actually fails or causes the stated problem
2. The correct example compiles and runs correctly
3. The rule doesn't contradict GORM or PostgreSQL official documentation
4. The rule isn't already covered by golang-best-practices-skill (check for overlap)

### Cross-Referencing golang-best-practices-skill

When adding a rule that extends or mirrors a rule in [golang-best-practices-skill](https://github.com/saifoelloh/golang-best-practices-skill), add a cross-reference note in the "Additional Context" section:

```markdown
> **Note from golang-best-practices-skill**: This rule relates to `{domain}/{rule-id}` in the companion Go skill. [Brief explanation of relationship].
```

Also update `metadata.json` `companion_skill.overlapping_rules` array.

### Style Guidelines

**Markdown**:
- Use `---` YAML frontmatter
- Use `## ` for top-level sections
- Use `### ` for subsections
- Use ` ```go ` for Go code blocks, ` ```sql ` for SQL blocks

**Code Examples**:
- `❌ BAD` examples should be realistic anti-patterns seen in real codebases
- `✅ GOOD` examples should be production-ready, including error handling
- Always include `ctx context.Context` in repository method signatures
- Use `fmt.Errorf("MethodName: %w", err)` pattern for error wrapping

**Tone**:
- Explain the *consequence*, not just the rule ("causes OOM crash" not just "don't do this")
- Focus on impact on PostgreSQL specifically, not generic database advice

## Pull Request Process

1. **Fork** the repository
2. **Create** a feature branch: `git checkout -b add-rule-{name}`
3. **Add** rule file following the template
4. **Update** `shared/_categories.md`, domain `SKILL.md`, root `SKILL.md`, `metadata.json`
5. **Commit**: `Add {priority}-{rule-name} to {domain}`
6. **Push** and open a Pull Request with:
   - Rule name and domain
   - Brief explanation and real-world example
   - Source citation

## Rule Quality Criteria

A good rule must:
- ✅ Be backed by authoritative source (GORM docs, PostgreSQL docs, published books)
- ✅ Address a real problem seen in Go + GORM + PostgreSQL production codebases
- ✅ Have clear detection criteria (developer knows how to spot it)
- ✅ Include working, realistic code examples
- ✅ Explain the PostgreSQL-specific impact
- ✅ Be actionable (developer knows exactly how to fix it)

Avoid:
- ❌ Generic ORM advice not specific to GORM or PostgreSQL
- ❌ Rules that duplicate golang-best-practices-skill without adding DB-specific value
- ❌ Overly nitpicky style rules with no correctness impact
- ❌ Rules without authoritative citation

## Questions?

Open an issue before starting work on:
- New domain skills (e.g., adding a `testing` or `monitoring` domain)
- Changes to existing CRITICAL rules
- Structural changes to the skill format

---

**Built with ❤️ for better Go database code**
