---
title: Usecases Orchestrate, Entities Decide
impact: ARCHITECTURE
impactDescription: Proper separation between coordination logic and business rules
tags: clean-architecture, usecase-pattern, domain-driven-design
source: Clean Architecture (Robert C. Martin) - Chapter 20
---

## Usecases Orchestrate, Entities Decide

Usecases coordinate workflow and call domain entities for business decisions. Domain entities contain business rules. Usecases should NOT contain complex business logic.

**Impact**: ARCHITECTURE - "Fat usecases" with business logic become hard to test, violate single responsibility, and duplicate logic across usecases.

### Detection

How to spot this issue in code:
- Usecase methods with complex `if/else` chains for business rules
- Calculations directly in usecase (not delegated to domain)
- Business rules duplicated across multiple usecases
- Domain entities are anemic (just data holders, no methods)
- Usecase contains domain knowledge

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - "Fat" usecase with business logic
package usecase

type OrderUsecase struct {
    orderRepo domain.OrderRepository
    userRepo  domain.UserRepository
}

func (u *OrderUsecase) CreateOrder(ctx context.Context, userID int64, items []Item) error {
    user, _ := u.userRepo.FindByID(ctx, userID)
    
    // BAD: Business logic in usecase
    var total float64
    for _, item := range items {
        total += item.Price * float64(item.Quantity)
    }
    
    // BAD: Business rules in usecase
    var discount float64
    if user.IsPremium {
        if total > 100 {
            discount = total * 0.15
        } else {
            discount = total * 0.10
        }
    } else {
        if total > 200 {
            discount = total * 0.05
        }
    }
    
    finalTotal := total - discount
    
    // BAD: Business rules in usecase
    status := "pending"
    if finalTotal > 1000000 {
        status = "needs_approval"
    }
    
    // BAD: Anemic domain entity (just a data holder)
    order := &domain.Order{
        UserID: userID,
        Total:  finalTotal,
        Status: status,
    }
    
    return u.orderRepo.Create(ctx, order)
}
```

**Why this is wrong**:
- Business rules scattered in usecase (hard to reuse)
- Cannot test business logic without usecase
- Usecase doing too much (violates SRP)
- Domain entities are anemic (no behavior)
- Business logic will be duplicated if another usecase needs same rules

### Correct (Best Practice)

```go
// ✅ GOOD - Usecase orchestrates, domain decides
package usecase

type OrderUsecase struct {
    orderRepo domain.OrderRepository
    userRepo  domain.UserRepository
}

func (u *OrderUsecase) CreateOrder(ctx context.Context, userID int64, items []domain.Item) (*domain.Order, error) {
    // GOOD: Orchestration - fetch dependencies
    user, err := u.userRepo.FindByID(ctx, userID)
    if err != nil {
        return nil, err
    }
    
    // GOOD: Domain entity contains business logic
    order := domain.NewOrder(user, items)
    
    // GOOD: Domain entity makes business decisions
    order.CalculateTotal()
    order.ApplyDiscount()
    order.DetermineStatus()
    
    // GOOD: Domain entity validates itself
    if err := order.Validate(); err != nil {
        return nil, err
    }
    
    // GOOD: Orchestration - persist result
    if err := u.orderRepo.Create(ctx, order); err != nil {
        return nil, err
    }
    
    // GOOD: Orchestration - trigger side effects
    if order.NeedsApproval() {
        u.notificationService.NotifyApprovalNeeded(ctx, order)
    }
    
    return order, nil
}
```

```go
// ✅ GOOD - Rich domain entity with business logic
package domain

type Order struct {
    ID       int64
    User     *User
    Items    []Item
    Subtotal float64
    Discount float64
    Total    float64
    Status   string
}

func NewOrder(user *User, items []Item) *Order {
    return &Order{
        User:  user,
        Items: items,
    }
}

func (o *Order) CalculateTotal() {
    o.Subtotal = 0
    for _, item := range o.Items {
        o.Subtotal += item.Price * float64(item.Quantity)
    }
}

func (o *Order) ApplyDiscount() {
    // GOOD: Business rules in domain entity
    if o.User.IsPremium {
        if o.Subtotal > 100 {
            o.Discount = o.Subtotal * 0.15
        } else {
            o.Discount = o.Subtotal * 0.10
        }
    } else {
        if o.Subtotal > 200 {
            o.Discount = o.Subtotal * 0.05
        }
    }
    
    o.Total = o.Subtotal - o.Discount
}

func (o *Order) DetermineStatus() {
    if o.Total > 1000000 {
        o.Status = "needs_approval"
    } else {
        o.Status = "pending"
    }
}

func (o *Order) NeedsApproval() bool {
    return o.Status == "needs_approval"
}

func (o *Order) Validate() error {
    if len(o.Items) == 0 {
        return errors.New("order must have at least one item")
    }
    if o.Total < 0 {
        return errors.New("invalid total")
    }
    return nil
}
```

**Why this is better**:
- Business logic centralized in domain entities (single source of truth)
- Domain entities are **rich** (behavior + data)
- Usecase is thin, focused on orchestration
- Easy to test business logic (just test domain entities)
- Business rules can be reused across multiple usecases

### Additional Context

**Usecase responsibilities** (Orchestration):
- Fetch data from repositories
- Call domain entities to make decisions
- Persist results
- Call external services
- Handle transactions
- Coordinate workflows

**Domain entity responsibilities** (Business Logic):
- Business rules and validations
- Calculations
- State transitions
- Business invariants
- Decision-making logic

**Anti-pattern: Anemic Domain Model**
```go
// BAD: Just a data holder
type Order struct {
    Total  float64
    Status string
}
// No methods, no behavior
```

**Good: Rich Domain Model**
```go
// GOOD: Has behavior
type Order struct {
    Total  float64
    Status string
}

func (o *Order) CalculateTotal() { ... }
func (o *Order) Validate() error { ... }
func (o *Order) CanBeCancelled() bool { ... }
```

**When usecase has logic**:
- Coordination logic (calling multiple services)
- Transaction management
- Authorization checks
- NOT business rules or calculations

**Related rules**:
- `arch-repository-business-logic` - Repositories do CRUD only
- `high-god-object` - Extract method for large usecases
- `medium-usecase-complexity` - Move business logic to domain entities

### References

- [Clean Architecture (Robert C. Martin)](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Anemic Domain Model (Martin Fowler)](https://martinfowler.com/bliki/AnemicDomainModel.html)
- [Domain-Driven Design (Eric Evans)](https://www.domainlanguage.com/ddd/)

---

**Rule Template Version**: 1.0  
**Last Updated**: 2026-01-22
