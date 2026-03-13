---
title: Check RowsAffected After Update and Delete Operations
impact: HIGH
impactDescription: Update/Delete on non-existent record returns nil error — silent no-op
tags: gorm, rows-affected, update, delete, silent-noop, not-found
source: GORM Docs - Update, GORM Docs - Delete
---

## Check RowsAffected After Update and Delete Operations

`db.Update`, `db.Delete`, and `db.Exec` return `nil` error when zero rows are matched — they succeeded at the SQL level but had no effect. Without checking `RowsAffected`, a "record not found" scenario is silently swallowed.

**Impact**: HIGH - Silent no-ops — update on deleted record returns success to caller

### Detection

- `db.Model(...).Update(...)` or `db.Delete(...)` without checking `result.RowsAffected`
- Repository methods that return `error` from Update/Delete but don't distinguish "not found" from "success"

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - no RowsAffected check
func (r *UserRepository) Deactivate(ctx context.Context, id uint) error {
    return r.db.WithContext(ctx).
        Model(&User{}).Where("id = ?", id).
        Update("is_active", false).Error
    // If id=999 doesn't exist: err=nil, but nothing was updated!
}
```

### Correct (Best Practice)

```go
// ✅ GOOD - check RowsAffected
func (r *UserRepository) Deactivate(ctx context.Context, id uint) error {
    result := r.db.WithContext(ctx).
        Model(&User{}).Where("id = ?", id).
        Updates(map[string]interface{}{"is_active": false})
    if result.Error != nil {
        return fmt.Errorf("UserRepository.Deactivate: %w", result.Error)
    }
    if result.RowsAffected == 0 {
        return ErrUserNotFound
    }
    return nil
}

// ✅ GOOD - same for Delete
func (r *SessionRepository) Delete(ctx context.Context, token string) error {
    result := r.db.WithContext(ctx).Where("token = ?", token).Delete(&Session{})
    if result.Error != nil {
        return result.Error
    }
    if result.RowsAffected == 0 {
        return ErrSessionNotFound
    }
    return nil
}
```

### References

- [GORM Docs: Update](https://gorm.io/docs/update.html)
- [GORM Docs: Delete](https://gorm.io/docs/delete.html)

---

**Rule Version**: 1.0
**Last Updated**: 2026-03-13
**Priority**: HIGH
**Auto-fixable**: No
