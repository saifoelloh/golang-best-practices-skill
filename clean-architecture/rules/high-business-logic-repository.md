---
title: No Business Logic in Repository Layer
impact: HIGH
impactDescription: Business logic in repositories makes it untestable and violates separation
tags: clean-architecture, repository, data-access, separation-of-concerns, gorm
source: Clean Architecture - Chapter 17 (Boundaries)
---

## No Business Logic in Repository Layer

Repositories should only handle CRUD operations and data access. Business rules, calculations, and complex logic belong in the usecase or domain layer.

**Impact**: HIGH - Violates Clean Architecture, logic buried in data layer

### Detection

How to spot this issue:
- Complex WHERE clauses with business rules
- Calculations in repository methods
- Status/state transitions in queries
- Pattern: Repository methods with `if`, `for`, business conditions

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Business logic in repository
func (r *EventRepo) GetAvailableEvents(ctx context.Context) ([]*Event, error) {
    var events []*Event
    
    // WRONG - Business rule: "available" logic
    err := r.db.WithContext(ctx).
        Where("start_date > ?", time.Now()).            // Business rule
        Where("status = ?", "published").               // Business rule
        Where("total_tickets > sold_tickets").          // Business rule
        Where("is_cancelled = ?", false).               // Business rule
        Find(&events).Error
    
    if err != nil {
        return nil, err
    }
    
    // WRONG - Filtering with business logic
    var available []*Event
    for _, event := range events {
        // WRONG - Calculation
        if event.TotalTickets-event.SoldTickets > 0 {
            available = append(available, event)
        }
    }
    
    return available, nil
}
```

**Why this is wrong**:
- Business rules buried in data layer
- Can't reuse logic elsewhere
- Hard to test business rules
- Violates Single Responsibility

### Correct (Best Practice)

```go
// ✅ GOOD - Repository does CRUD only
func (r *EventRepo) FindByFilter(ctx context.Context, filter EventFilter) ([]*Event, error) {
    query := r.db.WithContext(ctx)
    
    // Simple data filtering (no business rules)
    if filter.StartDateAfter != nil {
        query = query.Where("start_date > ?", filter.StartDateAfter)
    }
    
    if filter.Status != "" {
        query = query.Where("status = ?", filter.Status)
    }
    
    if filter.IsCancelled != nil {
        query = query.Where("is_cancelled = ?", *filter.IsCancelled)
    }
    
    var events []*Event
    err := query.Find(&events).Error
    return events, err
}

// ✅ Business logic in usecase
func (uc *EventUsecase) GetAvailableEvents(ctx context.Context) ([]*Event, error) {
    // Build filter based on business rules
    now := time.Now()
    cancelled := false
    filter := EventFilter{
        StartDateAfter: &now,
        Status:         "published",
        IsCancelled:    &cancelled,
    }
    
    // Get data from repository
    events, err := uc.eventRepo.FindByFilter(ctx, filter)
    if err != nil {
        return nil, err
    }
    
    // Apply business rule: check availability
    var available []*Event
    for _, event := range events {
        if event.IsAvailable() {  // Business method on entity
            available = append(available, event)
        }
    }
    
    return available, nil
}

// ✅ Business logic in domain entity
func (e *Event) IsAvailable() bool {
    return e.TotalTickets > e.SoldTickets
}
```

**Why this is better**:
- Clear separation of concerns
- Business logic in domain/usecase
- Repository is simple and focused
- Easy to test each layer

### Common Anti-Patterns

**Anti-Pattern 1: Calculations in Repository**

```go
// ❌ BAD - Price calculation in repo
func (r *OrderRepo) GetOrdersWithDiscount(ctx context.Context) ([]*Order, error) {
    var orders []*Order
    err := r.db.WithContext(ctx).Find(&orders).Error
    if err != nil {
        return nil, err
    }
    
    // WRONG - Business calculation
    for _, order := range orders {
        if order.TotalAmount > 1000000 {
            order.DiscountAmount = order.TotalAmount * 0.1  // 10% discount
        }
    }
    
    return orders, nil
}

// ✅ GOOD - Calculation in domain/usecase
type Order struct {
    TotalAmount float64
}

func (o *Order) CalculateDiscount() float64 {
    if o.TotalAmount > 1000000 {
        return o.TotalAmount * 0.1
    }
    return 0
}

func (uc *OrderUsecase) GetOrdersWithDiscount(ctx context.Context) ([]*Order, error) {
    orders, err := uc.orderRepo.FindAll(ctx)
    if err != nil {
        return nil, err
    }
    
    // Apply business logic
    for _, order := range orders {
        order.DiscountAmount = order.CalculateDiscount()
    }
    
    return orders, nil
}
```

**Anti-Pattern 2: Complex Business Queries**

```go
// ❌ BAD - Complex business logic in query
func (r *TicketRepo) GetExpiredUnpaidTickets(ctx context.Context) ([]*Ticket, error) {
    var tickets []*Ticket
    
    // WRONG - Multiple business rules in query
    err := r.db.WithContext(ctx).
        Joins("JOIN orders ON tickets.order_id = orders.id").
        Where("orders.status = ?", "pending").
        Where("orders.created_at < ?", time.Now().Add(-24*time.Hour)).  // Business rule: 24h expiry
        Where("tickets.is_used = ?", false).
        Find(&tickets).Error
    
    return tickets, err
}

// ✅ GOOD - Simple query, business logic in usecase
func (r *TicketRepo) FindByOrderID(ctx context.Context, orderID string) ([]*Ticket, error) {
    var tickets []*Ticket
    err := r.db.WithContext(ctx).
        Where("order_id = ?", orderID).
        Find(&tickets).Error
    return tickets, err
}

func (r *OrderRepo) FindByStatus(ctx context.Context, status string) ([]*Order, error) {
    var orders []*Order
    err := r.db.WithContext(ctx).
        Where("status = ?", status).
        Find(&orders).Error
    return orders, err
}

// Business logic in usecase
func (uc *TicketUsecase) GetExpiredUnpaidTickets(ctx context.Context) ([]*Ticket, error) {
    // Get pending orders
    orders, err := uc.orderRepo.FindByStatus(ctx, "pending")
    if err != nil {
        return nil, err
    }
    
    var expiredTickets []*Ticket
    expiryTime := time.Now().Add(-24 * time.Hour)  // Business rule
    
    for _, order := range orders {
        // Business rule: check if expired
        if order.CreatedAt.Before(expiryTime) {
            tickets, err := uc.ticketRepo.FindByOrderID(ctx, order.ID)
            if err != nil {
                return nil, err
            }
            
            // Business rule: filter unused
            for _, ticket := range tickets {
                if !ticket.IsUsed {
                    expiredTickets = append(expiredTickets, ticket)
                }
            }
        }
    }
    
    return expiredTickets, nil
}
```

**Anti-Pattern 3: State Transitions in Repository**

```go
// ❌ BAD - State transition logic in repo
func (r *OrderRepo) CancelExpiredOrders(ctx context.Context) error {
    // WRONG - Business logic about what to cancel
    return r.db.WithContext(ctx).
        Model(&Order{}).
        Where("status = ?", "pending").
        Where("created_at < ?", time.Now().Add(-24*time.Hour)).
        Update("status", "cancelled").
        Error
}

// ✅ GOOD - Simple update, logic in usecase
func (r *OrderRepo) UpdateStatus(ctx context.Context, orderID string, status string) error {
    return r.db.WithContext(ctx).
        Model(&Order{}).
        Where("id = ?", orderID).
        Update("status", status).
        Error
}

func (uc *OrderUsecase) CancelExpiredOrders(ctx context.Context) error {
    // Business logic: determine which orders to cancel
    orders, err := uc.orderRepo.FindByStatus(ctx, "pending")
    if err != nil {
        return err
    }
    
    expiryTime := time.Now().Add(-24 * time.Hour)
    
    for _, order := range orders {
        // Business rule
        if order.CreatedAt.Before(expiryTime) {
            if err := uc.orderRepo.UpdateStatus(ctx, order.ID, "cancelled"); err != nil {
                return err
            }
        }
    }
    
    return nil
}
```

### What Belongs in Repository

**✅ Allowed** (Data Access):
- CRUD operations (Create, Read, Update, Delete)
- Simple filtering by columns
- Sorting, pagination
- Joins for data retrieval
- Transactions (pass from usecase)
- Database-specific optimizations

**❌ Not Allowed** (Business Logic):
- Calculations
- Complex business rules
- State transitions based on business rules
- Data transformation for business needs
- Validation beyond data integrity

### Repository Pattern Template

```go
// ✅ Clean repository pattern
type EventRepository interface {
    Create(ctx context.Context, event *Event) error
    FindByID(ctx context.Context, id string) (*Event, error)
    FindAll(ctx context.Context) ([]*Event, error)
    FindByFilter(ctx context.Context, filter EventFilter) ([]*Event, error)
    Update(ctx context.Context, event *Event) error
    Delete(ctx context.Context, id string) error
}

// Implementation
type eventRepository struct {
    db *gorm.DB
}

func (r *eventRepository) FindByFilter(ctx context.Context, filter EventFilter) ([]*Event, error) {
    query := r.db.WithContext(ctx)
    
    // Simple column-based filtering only
    if filter.CityID != nil {
        query = query.Where("city_id = ?", *filter.CityID)
    }
    
    if filter.Status != nil {
        query = query.Where("status = ?", *filter.Status)
    }
    
    if filter.Limit > 0 {
        query = query.Limit(filter.Limit)
    }
    
    var events []*Event
    err := query.Find(&events).Error
    return events, err
}
```

### Quick Decision Tree

**Am I in a repository method?**
- Yes → **Is this a database operation?**
  - SELECT/INSERT/UPDATE/DELETE → **OK**
  - Calculation → **Move to usecase/domain**
  - Business rule → **Move to usecase/domain**
  - State transition → **Move to usecase**

**Checklist for repositories**:
- [ ] Only CRUD operations
- [ ] Simple WHERE clauses (column = value)
- [ ] No calculations or transformations
- [ ] No business rule validation
- [ ] No state transition logic

### References

- "Clean Architecture" by Robert C. Martin - Chapter 17
- [Repository Pattern in Go](https://threedots.tech/post/repository-pattern-in-go/)

---

**Rule Version**: 1.0  
**Last Updated**: 2026-01-21  
**Priority**: HIGH  
**Auto-fixable**: No (requires refactoring)
