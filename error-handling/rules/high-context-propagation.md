---
title: Propagate Context Through Call Chain
impact: HIGH
impactDescription: Breaking context chain prevents timeout/cancellation from working
tags: context, cancellation, timeout, propagation, call-chain
source: Learning Go - Chapter 14 (Context)
---

## Propagate Context Through Call Chain

Always accept and pass `context.Context` through your call chain. Creating new contexts breaks cancellation and timeout propagation.

**Impact**: HIGH - Timeouts and cancellations don't work, operations run indefinitely

### Detection

How to spot this issue:
- Functions creating `context.Background()` mid-chain
- Not accepting context parameter
- Not passing context to downstream calls
- Pattern: `ctx := context.Background()` in non-main functions

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Breaks context chain
func (s *Service) ProcessOrder(orderID int) error {
    // Creates new context! Breaks parent timeout
    ctx := context.Background()
    
    // Parent's timeout/cancellation ignored!
    return s.repo.GetOrder(ctx, orderID)
}

// ❌ BAD - Doesn't accept context
func (s *Service) FetchUserData(userID int) (*User, error) {
    // No way for caller to cancel or timeout!
    resp, err := http.Get(fmt.Sprintf("https://api.example.com/users/%d", userID))
    if err != nil {
        return nil, err
    }
    // ...
}

// ❌ BAD - Accepts context but doesn't use it
func (s *Service) SaveData(ctx context.Context, data *Data) error {
    // Ignores context! Uses fresh background
    return s.repo.Save(context.Background(), data)
}
```

**Why this is wrong**:
- Parent timeout doesn't propagate
- Cancellation signals lost
- Operations can't be stopped
- Wastes resources on cancelled requests

### Correct (Best Practice)

```go
// ✅ GOOD - Accepts and propagates context
func (s *Service) ProcessOrder(ctx context.Context, orderID int) error {
    // Uses provided context
    return s.repo.GetOrder(ctx, orderID)
}

// ✅ GOOD - Accepts context, passes to HTTP call
func (s *Service) FetchUserData(ctx context.Context, userID int) (*User, error) {
    url := fmt.Sprintf("https://api.example.com/users/%d", userID)
    
    req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
    if err != nil {
        return nil, err
    }
    
    resp, err := http.DefaultClient.Do(req)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    
    // ...
}

// ✅ GOOD - Propagates context through layers
func (h *Handler) GetUser(c echo.Context) error {
    // Extract from HTTP request
    ctx := c.Request().Context()
    
    // Pass to usecase
    user, err := h.usecase.GetUser(ctx, c.Param("id"))
    if err != nil {
        return err
    }
    
    return c.JSON(200, user)
}

func (uc *Usecase) GetUser(ctx context.Context, id string) (*User, error) {
    // Pass to repository
    return uc.repo.GetByID(ctx, id)
}

func (r *Repo) GetByID(ctx context.Context, id string) (*User, error) {
    var user User
    // Pass to database
    err := r.db.WithContext(ctx).First(&user, id).Error
    return &user, err
}
```

**Why this is better**:
- Timeout propagates through all layers
- Cancellation works end-to-end
- Can stop operations gracefully
- Resource-efficient

### When to Create New Context

```go
// ✅ ONLY create new context at entry points
func main() {
    ctx := context.Background()  // OK - entry point
    
    // Add timeout for entire operation
    ctx, cancel := context.WithTimeout(ctx, 30*time.Second)
    defer cancel()
    
    app.Run(ctx)
}

// ✅ Create derived context with additional timeout
func (s *Service) SlowOperation(ctx context.Context) error {
    // Add ADDITIONAL timeout for this specific operation
    ctx, cancel := context.WithTimeout(ctx, 10*time.Second)
    defer cancel()
    
    return s.repo.ComplexQuery(ctx)
}

// ❌ DON'T create fresh background context
func (s *Service) Operation(ctx context.Context) error {
    ctx = context.Background()  // WRONG! Loses parent context
    return s.repo.Query(ctx)
}
```

### Real-World Example

```go
// ❌ BAD - gRPC handler breaks context
func (s *EventGRPC) GetEvent(ctx context.Context, req *pb.GetEventRequest) (*pb.EventResponse, error) {
    // WRONG - Creates new context!
    newCtx := context.Background()
    
    // Client's timeout/cancellation ignored!
    event, err := s.usecase.GetEvent(newCtx, req.EventId)
    if err != nil {
        return nil, err
    }
    
    return toProto(event), nil
}

// Client sets 5s timeout
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

// This timeout is IGNORED by server!
resp, err := client.GetEvent(ctx, &pb.GetEventRequest{EventId: "123"})
```

```go
// ✅ GOOD - Proper context propagation
func (s *EventGRPC) GetEvent(ctx context.Context, req *pb.GetEventRequest) (*pb.EventResponse, error) {
    // Use provided context (has client's timeout)
    event, err := s.usecase.GetEvent(ctx, req.EventId)
    if err != nil {
        return nil, err
    }
    
    return toProto(event), nil
}

// Client's 5s timeout works!
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

resp, err := client.GetEvent(ctx, &pb.GetEventRequest{EventId: "123"})
// If GetEvent takes >5s, automatically cancelled ✅
```

### Database Query Example

```go
// ❌ BAD - GORM query without context
func (r *Repo) GetByID(id string) (*User, error) {
    var user User
    // No timeout! Could hang forever
    err := r.db.First(&user, id).Error
    return &user, err
}

// ✅ GOOD - GORM with context
func (r *Repo) GetByID(ctx context.Context, id string) (*User, error) {
    var user User
    // Respects context timeout/cancellation
    err := r.db.WithContext(ctx).First(&user, id).Error
    return &user, err
}
```

### HTTP Client Example

```go
// ❌ BAD - HTTP call without context
func fetchData(url string) ([]byte, error) {
    resp, err := http.Get(url)  // No timeout!
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    
    return io.ReadAll(resp.Body)
}

// ✅ GOOD - HTTP with context
func fetchData(ctx context.Context, url string) ([]byte, error) {
    req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
    if err != nil {
        return nil, err
    }
    
    resp, err := http.DefaultClient.Do(req)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    
    return io.ReadAll(resp.Body)
}
```

### Multiple Operations Pattern

```go
// ✅ GOOD - Sequential operations with same context
func (uc *Usecase) CreateOrder(ctx context.Context, req *CreateOrderRequest) error {
    // All operations share same context/timeout
    
    // Validate event
    event, err := uc.eventRepo.GetByID(ctx, req.EventID)
    if err != nil {
        return err
    }
    
    // Check availability
    available, err := uc.ticketRepo.CheckAvailability(ctx, req.EventID, req.Quantity)
    if err != nil {
        return err
    }
    
    if !available {
        return ErrSoldOut
    }
    
    // Create order
    order := &domain.Order{
        EventID:  req.EventID,
        Quantity: req.Quantity,
    }
    
    return uc.orderRepo.Create(ctx, order)
}
```

### context.Background vs context.TODO

```go
// ✅ Use context.Background() at entry points
func main() {
    ctx := context.Background()  // Known: starting fresh
    server.Start(ctx)
}

// ✅ Use context.TODO() when unsure/refactoring
func legacyFunction() {
    // TODO: Accept context parameter when refactoring
    ctx := context.TODO()
    newFunction(ctx)
}
```

### Additional Context

**Cancellation propagation**:

```go
// Parent cancelled → all children cancelled
ctx, cancel := context.WithCancel(context.Background())

go worker1(ctx)  // Gets same context
go worker2(ctx)  // Gets same context

cancel()  // Both workers receive cancellation signal
```

**Timeout propagation**:

```go
// Parent timeout: 10s
ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
defer cancel()

// Child can have shorter timeout
func operation(parentCtx context.Context) {
    ctx, cancel := context.WithTimeout(parentCtx, 5*time.Second)
    defer cancel()
    
    // Whichever timeout expires first cancels operation
}
```

### Quick Decision Tree

**Am I at an entry point (main, handler)?**
- Yes → **Create context.Background()**
- No → **Accept ctx context.Context parameter**

**Am I calling downstream function?**
- Yes → **Pass ctx to it**
- DB/HTTP call? → **Use WithContext(ctx)**

**Checklist**:
- [ ] Public functions accept `context.Context` as first param
- [ ] Context passed to all downstream calls
- [ ] No `context.Background()` in mid-chain
- [ ] DB queries use `WithContext(ctx)`
- [ ] HTTP requests use `NewRequestWithContext(ctx)`

### References

- [Go Blog: Context](https://go.dev/blog/context)
- "Learning Go" by Jon Bodner - Chapter 14
- [pkg.go.dev/context](https://pkg.go.dev/context)

---

**Rule Version**: 1.0  
**Last Updated**: 2026-01-21  
**Priority**: HIGH  
**Auto-fixable**: Partially (linter can detect missing context params)
