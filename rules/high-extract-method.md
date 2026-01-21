---
title: Extract Complex Logic to Named Methods
impact: HIGH
impactDescription: Complex inline logic reduces readability and increases bugs
tags: refactoring, extract-method, readability, naming
source: Refactoring (M. Fowler) - Ch 6; Clean Code (R. Martin) - Ch 3
---

## Extract Complex Logic to Named Methods

When you see complex code blocks with comments explaining what they do, that's a sign: the code itself should be self-documenting through well-named functions. Extracting these blocks dramatically improves readability and makes intent crystal clear.

**Impact**: HIGH - Improves readability, enables testing, reduces cognitive load

### Detection

When to extract a method:
- Code has explanatory comments ("Calculate tax", "Validate input")
- Nested conditionals >2 levels deep
- Code block that could be described in 3-5 words
- Repeated logic patterns (DRY violation)
- Difficulty understanding code at a glance
- Complex calculations inline

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Complex inline logic with explanatory comments
func ProcessWithdrawal(w *Withdrawal, event *Event) error {
    // Calculate progressive tax portion from total revenue
    // Using reverse calculation: tax = revenue * rate / (1 + rate)
    totalRevenue := w.RequestedAmount
    taxRate := 0.11
    var taxPortion int64
    if event.Tax != nil {
        if *event.Tax <= 100 {
            taxPortion = int64(float64(totalRevenue) * taxRate / (1.0 + taxRate))
        } else {
            taxPortion = int64(*event.Tax)
        }
    } else {
        taxPortion = int64(float64(totalRevenue) * 0.10 / 1.10)
    }
    
    // Calculate tier-based withdrawal fee
    // Tier 1: < 100M = 1%
    // Tier 2: 100M-500M = 0.8%
    // Tier 3: > 500M = 0.5%
    revenue := totalRevenue - taxPortion
    var fee int64
    if revenue < 100000000 {
        fee = revenue * 1 / 100
    } else if revenue < 500000000 {
        fee = revenue * 8 / 1000
    } else {
        fee = revenue * 5 / 1000
    }
    
    w.TaxPortion = taxPortion
    w.WithdrawalFee = fee
    w.AmountReceived = w.RequestedAmount - tax Portion - fee
    return nil
}
```

**Why this is wrong**:
- Comments explain WHAT code does (code should be obvious)
- Complex calculation logic inline (hard to test)
- Magic numbers without explanation (100000000, 8/1000)
- Cannot unit test tax or fee calculation independently
- High cognitive load to understand flow

### Correct (Best Practice)

``

`go
// ✅ GOOD - Extracted to well-named methods
func ProcessWithdrawal(w *Withdrawal, event *Event) error {
    taxPortion := calculateTaxPortion(w.RequestedAmount, event.Tax)
    revenue := w.RequestedAmount - taxPortion
    fee := calculateTierFee(revenue)
    
    w.TaxPortion = taxPortion
    w.WithdrawalFee = fee
    w.AmountReceived = w.RequestedAmount - taxPortion - fee
    return nil
}

// Method names document intent - no comments needed!
func calculateTaxPortion(amount int64, eventTax *int64) int64 {
    const defaultTaxRate = 0.10
    
    if eventTax == nil {
        return reverseCalculateTax(amount, defaultTaxRate)
    }
    
    if *eventTax <= 100 {
        rate := float64(*eventTax) / 100
        return reverseCalculateTax(amount, rate)
    }
    
    return int64(*eventTax)
}

func reverseCalculateTax(amount int64, rate float64) int64 {
    return int64(float64(amount) * rate / (1.0 + rate))
}

func calculateTierFee(revenue int64) int64 {
    const (
        tier1Threshold = 100_000_000 // 100M
        tier2Threshold = 500_000_000 // 500M
        tier1Rate      = 0.01         // 1%
        tier2Rate      = 0.008        // 0.8%
        tier3Rate      = 0.005        // 0.5%
    )
    
    if revenue < tier1Threshold {
        return int64(float64(revenue) * tier1Rate)
    }
    if revenue < tier2Threshold {
        return int64(float64(revenue) * tier2Rate)
    }
    return int64(float64(revenue) * tier3Rate)
}
```

**Why this is better**:
- Method names explain intent (no comments needed)
- Each function testable independently
- Constants replace magic numbers
- Easy to understand flow
- Low cognitive load
- Can reuse calculations elsewhere

### Additional Context

**When to extract:**
- Any time you feel compelled to write a comment
- When code block can be described in one sentence
- When you see "Step 1", "Step 2" comments
- When nested >2 levels
- When calculation is complex

**Naming guidelines:**
- Use verb phrases: `calculateTax`, `validateOrder`, `buildResponse`
- Be specific: `calculateTierFee` not `getFee`
- Avoid generic names: `processData`, `handleRequest`
- Length correlates with scope: longer names for wider scope

**Real example from today:**
```go
// Before (inline, 10 lines)
ticketPrice := int64(ticketType.Price)
if groupTicket != nil {
    ticketPrice = int64(groupTicket.Price)
}
// ...

// After (extracted)
ticketPrice := fc.getTicketPrice()
```

**Performance note:** No overhead - modern Go compiler inlines small functions.

### References

- [Refactoring.Guru - Extract Method](https://refactoring.guru/extract-method)
- [Refactoring.Guru - Comments Smell](https://refactoring.guru/smells/comments)
- Refactoring (Martin Fowler) - Chapter 6
- Clean Code (Robert C. Martin) - Chapter 3: "Functions"

---

**Rule Version**: 1.0  
**Last Updated**: 2026-01-22
