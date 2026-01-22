---
title: Repositories Do CRUD Only
impact: ARCHITECTURE
impactDescription: Prevents business logic leakage into data layer
tags: clean-architecture, repository-pattern, separation-of-concerns
source: Clean Architecture (Robert C. Martin) - Chapter 20
---

## Repositories Do CRUD Only

Repositories should only handle CRUD operations (Create, Read, Update, Delete) and database queries. No business logic, validations, or calculations.

**Impact**: ARCHITECTURE - Business logic in repositories makes it duplicated, hard to test, and violates single responsibility.

### Detection

How to spot this issue in code:
- Repository methods contain `if` statements for business rules
- Calculations or data transformations in repository
- Status changes or state management in repository
- Repository calling other services or repositories
- Validation logic in repository methods

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Repository with business logic
package repository

type OrderRepo struct {
    db *gorm.DB
}

func (r *OrderRepo) CreateOrder(ctx context.Context, order *domain.Order) error {
    // BAD: Validation in repository
    if order.Total < 0 {
        return errors.New("invalid total")
    }
    
    // BAD: Business rule in repository
    if order.Total > 1000000 {
        order.Status = "needs_approval"
    } else {
        order.Status = "approved"
    }
    
    // BAD: Calculations in repository
    order.Tax = order.Total * 0.1
    order.GrandTotal = order.Total + order.Tax
    
    // Finally saves (should be ONLY thing here)
    return r.db.Create(order).Error
}

func (r *OrderRepo) GetActiveOrders() ([]*domain.Order, error) {
    var orders []*domain.Order
    
    // BAD: Business logic defining "active"
    err := r.db.Where("status IN ? AND total > ?", 
        []string{"pending", "processing"}, 1000).Find(&orders).Error
    
    // BAD: Post-processing in repository
    for _, order := range orders {
        if order.CreatedAt.Before(time.Now().Add(-24 * time.Hour)) {
            order.IsPriority = true
        }
    }
    
    return orders, err
}
```

**Why this is wrong**:
- Business rules scattered across layers (hard to find and change)
- Cannot test business logic without database
- Violates single responsibility principle
- Repository doing more than data persistence

### Correct (Best Practice)

```go
// ✅ GOOD - Repository does CRUD only
package repository

type OrderRepo struct {
    db *gorm.DB
}

func (r *OrderRepo) Create(ctx context.Context, order *domain.Order) error {
    return r.db.Create(order).Error // GOOD: Simple CRUD
}

func (r *OrderRepo) FindByID(ctx context.Context, id int64) (*domain.Order, error) {
    var order domain.Order
    err := r.db.First(&order, id).Error
    return &order, err
}

func (r *OrderRepo) FindByStatusIn(ctx context.Context, statuses []string) ([]*domain.Order, error) {
    var orders []*domain.Order
    err := r.db.Where("status IN ?", statuses).Find(&orders).Error
    return orders, err // GOOD: Just returns data, no business logic
}

func (r *OrderRepo) Update(ctx context.Context, order *domain.Order) error {
    return r.db.Save(order).Error
}
```

```go
// ✅ GOOD - Business logic in usecase and domain
package usecase

type OrderUsecase struct {
    orderRepo domain.OrderRepository
}

func (u *OrderUsecase) CreateOrder(ctx context.Context, req *CreateOrderRequest) (*domain.Order, error) {
    // GOOD: Create domain entity
    order := domain.NewOrder(req.Items, req.Total)
    
    // GOOD: Validation in domain or usecase
    if err := order.Validate(); err != nil {
        return nil, err
    }
    
    // GOOD: Business rules in domain entity
    order.CalculateTotals()
    
    // GOOD: Business logic in usecase
    if order.NeedsApproval() {
        order.Status = "needs_approval"
    } else {
        order.Status = "approved"
    }
    
    // Repository just persists
    if err := u.orderRepo.Create(ctx, order); err != nil {
        return nil, err
    }
    
    return order, nil
}

func (u *OrderUsecase) GetActiveOrders(ctx context.Context) ([]*domain.Order, error) {
    // GOOD: Usecase defines what "active" means
    statuses := []string{"pending", "processing"}
    orders, err := u.orderRepo.FindByStatusIn(ctx, statuses)
    if err != nil {
        return nil, err
    }
    
    // GOOD: Post-processing in usecase
    for _, order := range orders {
        order.DeterminePriority() // Domain method
    }
    
    return orders, nil
}
```

```go
// ✅ GOOD - Domain entity with business logic
package domain

type Order struct {
    ID         int64
    Total      float64
    Tax        float64
    GrandTotal float64
    Status     string
    CreatedAt  time.Time
}

func (o *Order) Validate() error {
    if o.Total < 0 {
        return errors.New("invalid total")
    }
    return nil
}

func (o *Order) CalculateTotals() {
    o.Tax = o.Total * 0.1
    o.GrandTotal = o.Total + o.Tax
}

func (o *Order) NeedsApproval() bool {
    return o.Total > 1000000
}

func (o *Order) DeterminePriority() {
    o.IsPriority = o.CreatedAt.Before(time.Now().Add(-24 * time.Hour))
}
```

**Why this is better**:
- Single responsibility: repository only does data access
- Business logic centralized in domain/usecase layer
- Easy to test business logic without database
- Clear separation of concerns

### Additional Context

**Repository responsibilities** (ONLY):
- CRUD operations
- Database queries
- Data mapping (DB model ↔ domain entity)
- Transaction handling (coordinated by usecase)

**NOT repository responsibilities**:
- Validation
- Business rules
- Calculations
- State transitions
- Calling other services

**When queries get complex**:
Provide flexible repository methods that accept parameters:
```go
type OrderQueryParams struct {
    Statuses []string
    MinTotal float64
    MaxTotal float64
}

func (r *OrderRepo) Find(ctx context.Context, params OrderQueryParams) ([]*domain.Order, error) {
    query := r.db
    if len(params.Statuses) > 0 {
        query = query.Where("status IN ?", params.Statuses)
    }
    // ... apply other filters
    return orders, query.Find(&orders).Error
}
```

**Related rules**:
- `high-business-logic-repository` - No business logic in data layer
- `arch-usecase-orchestration` - Usecases orchestrate, entities decide

### References

- [Clean Architecture (Robert C. Martin)](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Repository Pattern](https://martinfowler.com/eaaCatalog/repository.html)
- [Domain-Driven Design](https://www.domainlanguage.com/ddd/)

---

**Rule Template Version**: 1.0  
**Last Updated**: 2026-01-22
