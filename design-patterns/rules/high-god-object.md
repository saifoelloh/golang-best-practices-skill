---
title: Extract Logic from God Objects
impact: HIGH
impactDescription: Functions >300 lines become unmaintainable and untestable
tags: refactoring, god-object, extract-method, maintainability
source: Refactoring (M. Fowler) - Ch 3; Clean Code (R. Martin) - Ch 3
---

## Extract Logic from God Objects

Functions or structs that grow beyond 300 lines violate the Single Responsibility Principle and become maintenance nightmares. Breaking them into focused, cohesive pieces dramatically improves readability, testability, and team velocity.

**Impact**: HIGH - Decreases maintainability, increases bugs, blocks collaboration

### Detection

How to identify god objects/functions:
- Functions >200 lines (yellow flag)
- Functions >300 lines (red flag - refactor immediately)
- Functions handling 5+ distinct responsibilities
- Comments like "Step 1:", "Step 2:" indicating separate concerns
- Difficulty writing unit tests for the function
-

 Multiple levels of nested blocks (>3 levels)

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - God function doing everything (500+ lines!)
func CreateTransactionByOrder(ctx context.Context, orderId, paymentMethodId string) (*Transaction, error) {
    // Step 1: Validate order (50 lines)
    order, err := u.OrderRepo.FindById(orderId)
    if err != nil {
        return nil, err
    }
    if order.Status != "pending" {
        return nil, errors.New("invalid status")
    }
    // ... 40 more validation lines
    
    // Step 2: Calculate fees (195 lines!)
    ticketPrice := int64(ticketType.Price)
    if groupTicket != nil {
        ticketPrice = int64(groupTicket.Price)
    }
    originalPrice := ticketPrice * int64(order.Quantity)
    basePrice := originalPrice
    
    // ... 190 more fee calculation lines
    
    // Step 3-6: More responsibilities (300+ lines)
    // ... payment, xendit, email, tickets
    
    return transaction, nil
}
```

**Why this is wrong**:
- Impossible to understand without reading entire function
- Cannot test fee calculation independently
- Changes to email logic risk breaking payment processing
- Multiple developers cannot work on different aspects
- High cyclomatic complexity (>50)
- Violates Single Responsibility Principle

### Correct (Best Practice)

```go
// ✅ GOOD - Extracted to focused functions and domain entities
func CreateTransactionByOrder(ctx context.Context, orderId, paymentMethodId string) (*Transaction, error) {
    order, err := u.validateAndGetOrder(ctx, orderId)
    if err != nil {
        return nil, fmt.Errorf("order validation: %w", err)
    }
    
    feeCalc, err := u.calculateFees(ctx, order, paymentMethodId)
    if err != nil {
        return nil, fmt.Errorf("fee calculation: %w", err)
    }
    
    payment, err := u.createPayment(ctx, order, feeCalc)
    if err != nil {
        return nil, fmt.Errorf("payment creation: %w", err)
    }
    
    if err := u.processPaymentProvider(ctx, payment); err != nil {
        return nil, fmt.Errorf("payment processing: %w", err)
    }
    
    tickets, err := u.createTickets(ctx, order, payment)
    if err != nil {
        return nil, fmt.Errorf("ticket creation: %w", err)
    }
    
    u.sendNotificationsAsync(ctx, order, payment)
    
    return u.buildTransaction(order, payment, tickets

), nil
}

// Fee calculation extracted to domain entity
func (u *OrderUsecase) calculateFees(ctx context.Context, order *Order, paymentMethodId string) (*FeeCalculationJSON, error) {
    event, ticketType, paymentMethod, err := u.getRequiredEntities(ctx, order, paymentMethodId)
    if err != nil {
        return nil, err
    }
    
    // Business logic moved to domain
    calculator := domain.NewFeeCalculator(event, ticketType, paymentMethod, order)
    if order.GroupTicket != nil {
        calculator = calculator.WithGroupTicket(order.GroupTicket)
    }
    
    return calculator.Calculate()
}
```

**Why this is better**:
- Each function has single, clear responsibility
- Can test fee calculation independently
- Easy to understand orchestration flow
- Multiple developers can work on different aspects
- Easy to modify one aspect without affecting others
- Follows Clean Architecture - domain logic in domain layer

### Additional Context

**When to extract:**
- Function >50 lines: Consider extraction
- Function >100 lines: Strongly consider
- Function >200 lines: MUST extract
- Function >300 lines: Critical technical debt

**Extraction strategies:**
1. **Extract Method** - Pull out coherent blocks
2. **Extract Class** - Move related methods to new struct
3. **Move to Domain** - Business logic belongs in domain entities
4. **Introduce Parameter Object** - Group related parameters

**Real-world example:**
Reduced `CreateTransactionByOrder` from 500+ lines to ~30 lines by extracting 195-line fee calculation to `domain.FeeCalculator`.

### References

- [Refactoring.Guru - Long Method](https://refactoring.guru/smells/long-method)
- [Refactoring.Guru - Extract Method](https://refactoring.guru/extract-method)
- Refactoring (Martin Fowler) - Chapter 3
- Clean Code (Robert C. Martin) - Chapter 3

---

**Rule Version**: 1.0  
**Last Updated**: 2026-01-22
