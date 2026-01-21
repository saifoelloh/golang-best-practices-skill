---
title: Move Business Logic to Domain Entities
impact: MEDIUM
impactDescription: Fat usecases become hard to test and maintain
tags: clean-architecture, domain-model, usecase, entity, complexity
source: Clean Architecture - Domain Layer
---

## Move Business Logic to Domain Entities

Business logic belongs in domain entities, not usecases. Usecases should orchestrate, entities should decide. This prevents fat usecases and promotes rich domain models.

**Impact**: MEDIUM - Untestable logic, low cohesion

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Business logic in usecase
func (uc *OrderUsecase) ValidateOrder(order *Order) error {
    // Business rules scattered in usecase
    if order.Quantity <= 0 {
        return errors.New("quantity must be positive")
    }
    if order.TotalPrice != order.Quantity * order.UnitPrice {
        return errors.New("price mismatch")
    }
    // ...100 more lines of validation
}
```

### Correct (Best Practice)

```go
// ✅ GOOD - Business logic in entity
func (o *Order) Validate() error {
    if o.Quantity <= 0 {
        return errors.New("quantity must be positive")
    }
    if !o.IsPriceCorrect() {
        return errors.New("price mismatch")
    }
    return nil
}

func (o *Order) IsPriceCorrect() bool {
    return o.TotalPrice == o.Quantity * o.UnitPrice
}

// Usecase just orchestrates
func (uc *OrderUsecase) CreateOrder(order *Order) error {
    if err := order.Validate(); err != nil {
        return err
    }
    return uc.repo.Save(order)
}
```

---

**Rule Version**: 1.0  
**Priority**: MEDIUM
