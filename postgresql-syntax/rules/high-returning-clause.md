---
title: Use RETURNING to Get Generated Values After Insert/Update
impact: HIGH
impactDescription: Second SELECT after INSERT creates race condition window and doubles round-trips
tags: postgresql, returning, insert, update, atomicity, round-trip
source: PostgreSQL Docs - RETURNING Clause
---

## Use RETURNING to Get Generated Values After Insert/Update

A common pattern is to `INSERT` a record then immediately `SELECT` it back to get the generated `id`, `created_at`, or other server-side defaults. This creates a race condition window and doubles the number of DB round-trips. PostgreSQL's `RETURNING` clause retrieves generated values atomically in the same statement.

**Impact**: HIGH - Race condition between INSERT and SELECT, unnecessary extra round-trip

### Detection

How to spot this issue:
- `db.Create(&model)` followed by `db.First(&created, "field = ?", value)` to get generated fields
- `db.Exec("INSERT INTO ...")` followed by `db.Raw("SELECT id FROM ...")` in the same function
- Any pattern of write + immediate read of the same record

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - two round-trips, race condition between insert and select
func (r *UserRepository) Create(ctx context.Context, user *User) (*User, error) {
    if err := r.db.WithContext(ctx).Create(user).Error; err != nil {
        return nil, err
    }
    // Another insert could happen between these two queries
    var created User
    if err := r.db.WithContext(ctx).First(&created, "email = ?", user.Email).Error; err != nil {
        return nil, err
    }
    return &created, nil
}
```

### Correct (Best Practice)

```go
// ✅ GOOD - GORM Create already uses RETURNING internally — ID is populated on the struct
func (r *UserRepository) Create(ctx context.Context, user *User) error {
    // After this call, user.ID and user.CreatedAt are populated by PostgreSQL
    return r.db.WithContext(ctx).Create(user).Error
}

// ✅ GOOD - explicit RETURNING for raw INSERT
func (r *UserRepository) CreateRaw(ctx context.Context, name, email string) (*User, error) {
    var user User
    err := r.db.WithContext(ctx).Raw(`
        INSERT INTO users (name, email, created_at)
        VALUES (?, ?, NOW())
        RETURNING id, name, email, created_at
    `, name, email).Scan(&user).Error
    if err != nil {
        return nil, fmt.Errorf("UserRepository.CreateRaw: %w", err)
    }
    return &user, nil
}

// ✅ GOOD - RETURNING after UPDATE
func (r *UserRepository) UpdateAndReturn(ctx context.Context, id uint, name string) (*User, error) {
    var user User
    err := r.db.WithContext(ctx).Raw(`
        UPDATE users SET name = ?, updated_at = NOW()
        WHERE id = ?
        RETURNING id, name, email, updated_at
    `, name, id).Scan(&user).Error
    return &user, err
}
```

**Why this is better**:
- Atomic: generated values read in the same transaction as the write
- Single round-trip instead of two
- No race condition: impossible for another write to intervene

### References

- [PostgreSQL Docs: RETURNING Clause](https://www.postgresql.org/docs/16/dml-returning.html)
- [GORM Docs: Create](https://gorm.io/docs/create.html)

---

**Rule Version**: 1.0
**Last Updated**: 2026-03-13
**Priority**: HIGH
**Auto-fixable**: No
