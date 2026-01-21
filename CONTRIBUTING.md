# Contributing to Golang Best Practices Skill

Thank you for considering contributing! This skill is designed to help developers write better Go code.

## How to Contribute

### Adding a New Rule

1. **Research** - Ensure the rule is backed by authoritative source
   - Prefer: Official Go documentation, published books, established style guides
   - Avoid: Personal opinions without citations

2. **Use the Template** - Start with `rules/_template.md`
   ```bash
   cp rules/_template.md rules/[priority]-[name].md
   ```

3. **Fill in Required Sections**:
   - YAML frontmatter (title, impact, tags, source)
   - Clear explanation of why it matters
   - Detection criteria
   - Incorrect example with explanation
   - Correct example with explanation
   - References

4. **Prioritize Correctly**:
   - **CRITICAL**: Production bugs, crashes, security issues
   - **HIGH**: Reliability, architecture, correctness
   - **MEDIUM**: Code quality, idioms, maintainability

5. **Add to Index**:
   - Update `rules/_categories.md`
   - Update `SKILL.md` quick reference
   - Update `metadata.json` statistics

### Testing Your Rule

Before submitting, test your rule:

1. Find real code examples that violate the rule
2. Verify the "correct" example actually solves the problem
3. Ensure the rule doesn't create false positives

### Code Examples

- ❌ **Bad examples** should show real anti-patterns
- ✅ **Good examples** should be production-ready
- Include comments explaining WHY, not just WHAT
- Keep examples concise but complete

### Style Guidelines

**Markdown**:
- Use `---` for YAML frontmatter
- Use `## ` for section headers
- Use ` ```go ` for code blocks
- Use **bold** for emphasis, *italic* for notes

**Code**:
- Follow Go style guide
- Use meaningful variable names
- Include error handling
- Add comments for clarity

**Tone**:
- Be helpful, not judgmental
- Explain the reasoning
- Focus on impact, not rules for rules' sake

## Pull Request Process

1. **Fork** the repository
2. **Create** a feature branch (`git checkout -b add-rule-x`)
3. **Add** your rule following the template
4. **Update** documentation (categories, SKILL.md, metadata.json)
5. **Test** with real code examples
6. **Commit** with clear message: `Add [priority]-[name] rule`
7. **Push** to your fork
8. **Submit** a Pull Request with:
   - Rule name and priority
   - Brief explanation
   - Source citation
   - Example violation (if possible)

## Rule Criteria

A good rule should:

- ✅ Be backed by authoritative source
- ✅ Solve a real problem seen in production
- ✅ Have clear detection criteria
- ✅ Include working code examples
- ✅ Explain the impact/reasoning
- ✅ Be actionable (developer knows how to fix)

Avoid:
- ❌ Personal preferences without backing
- ❌ Overly broad/vague rules
- ❌ Rules that contradict Go idioms
- ❌ Nitpicky style rules (use gofmt for that)

## Questions?

Open an issue for discussion before starting work on:
- Major new rule categories
- Changes to existing critical rules
- Structural changes to the skill

## Code of Conduct

- Be respectful and constructive
- Focus on code, not people
- Cite sources for disagreements
- Assume good intent

Thank you for helping make Go code better! 🚀
