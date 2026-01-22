---
title: Transactions Belong in Usecase Layer
impact: HIGH
impactDescription: Transactions in repository prevent multi-repository atomic operations
tags: transaction, usecase, repository, gorm, atomic-operations
source: Clean Architecture - Application Business Rules

---

## Transactions Belong in Usecase Layer

Transaction management should be in the usecase layer, not repository. This allows atomic operations across multiple repositories.

**Impact**: HIGH - Can't compose atomic multi-repo operations

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Transaction in repository
func (r *OrderRepo) CreateWithItems(order *Order) error {
    return r.db.Transaction(func(tx *gorm.DB) error {
        if err := tx.Create(order).Error; err != nil {
            return err
        }
        // Can't call other repos!
        return nil
    })
}
```

### Correct (Best Practice)

```go
// ✅ GOOD - Transaction in usecase
func (uc *OrderUsecase) CreateOrder(ctx context.Context, order *Order) error {
    return uc.db.Transaction(func(tx *gorm.DB) error {
        orderRepo := repository.NewOrderRepository(tx)
        itemRepo := repository.NewItemRepository(tx)
        
        if err := orderRepo.Create(ctx, order); err != nil {
            return err
        }
        
        for _, item := range order.Items {
            if err := itemRepo.Create(ctx, item); err != nil {
                return err
            }
        }
        
        return nil
    })
}
```

---

**Rule Version**: 1.0  
**Priority**: HIGH
