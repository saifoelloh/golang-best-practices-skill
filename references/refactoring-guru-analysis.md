# Refactoring.Guru Analysis

**Source:** https://refactoring.guru  
**Analysis Date:** 2026-01-22  
**Purpose:** Compare with book research and identify unique contributions

---

## 🎯 Overview

Refactoring.guru is an **EXCELLENT** modern resource that:
- Visualizes Fowler's Refactoring catalog
- Explains GoF Design Patterns
- Organizes code smells by category
- Provides multiple language examples
- Uses clear diagrams and visuals

**Key Strength:** More accessible than the original books, perfect for learning!

---

## 📊 Coverage Comparison

### What refactoring.guru ADDS beyond the 5 books:

**1. Better Organization**
- Code smells grouped into 5 clear categories:
  - Bloaters
  - Object-Orientation Abusers
  - Change Preventers
  - Dispensables
  - Couplers

**2. Visual Diagrams**
- UML diagrams for each pattern
- Before/after refactoring visuals
- Clear step-by-step illustrations

**3. Multiple Languages**
- Examples in Java, C#, PHP, Python, TypeScript
- (Unfortunately no Go examples, but patterns translate)

**4. Relationships Map**
- Shows which refactorings fix which smells
- Pattern relationships clearly mapped

---

## 🔍 Code Smells from Refactoring.Guru

### Bloaters (Things that grow too big)

| Smell | Our Plan Covers? | Rule Name |
|-------|------------------|-----------|
| **Long Method** | ✅ YES | high-god-object, high-extract-method |
| **Large Class** | ✅ YES | high-god-object |
| **Primitive Obsession** | ✅ YES | medium-primitive-obsession |
| **Long Parameter List** | ✅ YES | medium-long-parameter-list |
| **Data Clumps** | ✅ YES | medium-data-clumps |

**Coverage: 100%** ✅

### Object-Orientation Abusers

| Smell | Our Plan Covers? | Notes |
|-------|------------------|-------|
| Switch Statements | ⚠️ PARTIAL | Could add medium-switch-to-polymorphism |
| Temporary Field | ❌ NO | Less relevant in Go (no instance vars) |
| Refused Bequest | ❌ NO | Not applicable (Go has no inheritance) |
| Alternative Classes with Different Interfaces | ⚠️ PARTIAL | Covered by interface-segregation |

**Coverage: 50%** - Some not applicable to Go

### Change Preventers

| Smell | Our Plan Covers? | Notes |
|-------|------------------|-------|
| Divergent Change | ⚠️ IMPLIED | Covered by SRP in high-god-object |
| Shotgun Surgery | ⚠️ IMPLIED | Result of good decomposition |
| Parallel Inheritance Hierarchies | ❌ NO | Not applicable (Go has no inheritance) |

**Coverage: ~50%** - Not directly, but principles covered

### Dispensables (Unnecessary things)

| Smell | Our Plan Covers? | Notes |
|-------|------------------|-------|
| Comments | ⚠️ IMPLIED | Extract method makes code self-documenting |
| Duplicate Code | ⚠️ IMPLIED | DRY principle in extract-method |
| Lazy Class | ❌ NO | Could add if observed in practice |
| Dead Code | ❌ NO | Linter handles this |
| Speculative Generality | ❌ NO | YAGNI principle, less critical |

**Coverage: ~30%** - Covered indirectly

### Couplers (Excessive coupling)

| Smell | Our Plan Covers? | Rule Name |
|-------|------------------|-----------|
| **Feature Envy** | ✅ YES | medium-feature-envy |
| **Inappropriate Intimacy** | ⚠️ PARTIAL | Related to feature-envy |
| Message Chains | ❌ NO | Law of Demeter - could add |
| Middle Man | ❌ NO | Less common in Go's flat style |

**Coverage: 50%** - Key ones covered

---

## 🎨 Design Patterns from Refactoring.Guru

### Creational Patterns

| Pattern | Our Plan Covers? | Rule Name |
|---------|------------------|-----------|
| **Factory Method** | ✅ YES | medium-factory-constructor |
| **Abstract Factory** | ❌ NO | Less common in Go |
| **Builder** | ✅ YES | medium-builder-pattern |
| **Prototype** | ❌ NO | Rare in Go |
| **Singleton** | ⚠️ IMPLIED | sync.Once pattern (could add) |

**Coverage: 40%** - Most important ones covered

### Structural Patterns

| Pattern | Our Plan Covers? | Notes |
|---------|------------------|-------|
| Adapter | ❌ NO | Could add as medium priority |
| Bridge | ❌ NO | Less common |
| Composite | ❌ NO | Less common |
| **Decorator** | ⚠️ IMPLIED | Middleware pattern (could add) |
| Facade | ⚠️ IMPLIED | Usecase layer is facade |
| Flyweight | ❌ NO | Optimization pattern |
| Proxy | ❌ NO | Less common |

**Coverage: ~20%** - Could add Decorator for middleware

### Behavioral Patterns

| Pattern | Our Plan Covers? | Notes |
|---------|------------------|-------|
| Chain of Responsibility | ⚠️ IMPLIED | Middleware chains |
| Command | ❌ NO | Could add if needed |
| Iterator | ❌ NO | Built into Go (for/range) |
| Mediator | ❌ NO | Less common |
| Memento | ❌ NO | Less common |
| Observer | ❌ NO | Channels handle this |
| State | ❌ NO | Could add if needed |
| **Strategy** | ⚠️ IMPLIED | Interface-based (could formalize) |
| Template Method | ❌ NO | Not applicable (no inheritance) |
| Visitor | ❌ NO | Less common |

**Coverage: ~15%** - Go handles many via interfaces/channels

---

## 💡 Unique Value of Refactoring.Guru

### 1. Smell → Refactoring → Pattern Flow

Refactoring.guru shows the **relationships**:

```
Long Method (smell)
    ↓
Extract Method (refactoring)
    ↓
Strategy Pattern (pattern)
```

**Our Skill Should Add:** Clear "fixes" section in each rule!

### 2. When to Apply / When NOT to Apply

Each pattern/refactoring has:
- ✅ Pros
- ❌ Cons
- ⚠️ When to use
- 🚫 When NOT to use

**Our Skill Lacks:** This nuance! We should add it.

### 3. Step-by-Step How

Refactoring.guru provides:
1. Initial code
2. Step 1: Do X
3. Step 2: Do Y
4. Final code

**Our Skill Should Consider:** Adding "How to Fix" steps

---

## 🎯 Recommendations for Our Skill

### Should We ADD from Refactoring.Guru?

**✅ Definitely Add (3 rules):**

1. **medium-switch-to-strategy**
   - Common in Go (type switches)
   - Refactoring.guru has great examples
   - Replace type switch with interfaces

2. **medium-middleware-decorator**
   - Extremely common in Go (http.Handler)
   - Not explicitly covered yet
   - Refactoring.guru calls it "Decorator"

3. **medium-law-of-demeter**
   - Message chains / inappropriate intimacy
   - Reduce coupling
   - "Don't talk to strangers"

**⚠️ Maybe Add (2 rules):**

4. **medium-singleton-once**
   - sync.Once pattern (Go-specific)
   - Common enough to warrant rule

5. **low-duplicate-code**
   - Extract method handles this
   - But could be explicit

**❌ Don't Add:**
- Inheritance-based patterns (not Go)
- Rare patterns (Memento, Visitor, etc.)
- Patterns Go handles natively (Iterator)

---

## 📊 Updated Rule Count

**Current Plan:** 10 rules  
**Recommended Additions:** +3 rules  
**New Total:** 13 rules

### Updated List:

**HIGH Priority (2):**
1. high-god-object
2. high-extract-method

**MEDIUM Priority (11):**
3. medium-primitive-obsession
4. medium-long-parameter-list
5. medium-data-clumps
6. medium-feature-envy
7. medium-magic-constants
8. medium-builder-pattern
9. medium-factory-constructor
10. medium-introduce-parameter-object
11. **medium-switch-to-strategy** ⭐ NEW
12. **medium-middleware-decorator** ⭐ NEW
13. **medium-law-of-demeter** ⭐ NEW

---

## ✅ Coverage Summary

### What Our 13 Rules Cover:

**From Fowler's Book:** ~80% of relevant refactorings  
**From GoF Patterns:** ~40% (Go-relevant ones: ~70%)  
**From Uncle Bob:** ~90% of key principles  
**From Refactoring.Guru Smells:** ~70% of Bloaters, ~50% of Couplers  

**Overall Coverage:** ~70% of relevant patterns for Go**

**Missing:** Mostly inheritance-based or rare patterns not applicable to Go

---

## 🎯 Final Verdict

**Is Refactoring.Guru Covered by the 5 Books?**

**YES, mostly** - It's based on Fowler + GoF:
- Code smells = Fowler's catalog
- Refactorings = Fowler's techniques  
- Patterns = GoF + some extras

**BUT it adds unique value:**
- ✅ Better visual organization
- ✅ Clear smell → refactoring → pattern flow
- ✅ When to use / when NOT to use
- ✅ Step-by-step instructions
- ✅ Multiple language examples

**Recommendation:**
1. ✅ Keep our 10 planned rules
2. ✅ Add 3 more from refactoring.guru analysis (Switch, Decorator, Law of Demeter)
3. ✅ Cite refactoring.guru as additional reference
4. ✅ Use their organizational structure for our rules

**Total: 13 rules for v1.1.0** 🎯

---

## 📚 Citations

Should add to each relevant rule:

```markdown
**Sources:**
- Refactoring (Fowler): Chapter X
- Clean Code (Martin): Chapter Y
- Refactoring.Guru: [URL]
```

This gives users multiple learning paths!

---

**Last Updated:** 2026-01-22  
**Recommendation:** PROCEED with 13 rules (10 original + 3 from refactoring.guru)
