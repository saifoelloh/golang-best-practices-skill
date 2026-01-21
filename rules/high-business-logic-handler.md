---
title: Keep Delivery Layer Thin
impact: HIGH
impactDescription: Business logic in handlers violates Clean Architecture, making code untestable
tags: clean-architecture, grpc, http, handler, delivery, separation-of-concerns
source: Clean Architecture - Chapter 22 (The Clean Architecture)
---

## Keep Delivery Layer Thin

Delivery layer (gRPC/HTTP handlers) should only handle protocol concerns: validation, marshaling, and delegating to usecases. Business logic belongs in the usecase or domain layer.

**Impact**: HIGH - Violates Clean Architecture, untestable business logic

### Detection

How to spot this issue:
- Complex business logic in handler functions
- Database queries in handlers
- Calculations, transformations in delivery layer
- Pattern: `func (h *Handler) GetUser()` contains `if`, `for`, business rules

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Business logic in gRPC handler
func (s *EventGRPC) GetEvent(ctx context.Context, req *pb.GetEventRequest) (*pb.EventResponse, error) {
    // Input validation - OK
    if req.EventId == "" {
        return nil, status.Error(codes.InvalidArgument, "event_id required")
    }
    
    // WRONG - Database query in handler!
    var event domain.Event
    if err := s.db.Where("id = ?", req.EventId).First(&event).Error; err != nil {
        return nil, status.Error(codes.NotFound, "event not found")
    }
    
    // WRONG - Business logic in handler!
    // Check if event is available
    if time.Now().Before(event.StartDate) {
        event.Status = "upcoming"
    } else if time.Now().After(event.EndDate) {
        event.Status = "finished"
    } else {
        event.Status = "ongoing"
    }
    
    // WRONG - Calculating ticket availability
    var soldCount int64
    s.db.Model(&domain.Ticket{}).Where("event_id = ?", event.Id).Count(&soldCount)
    availableTickets := event.TotalTickets - int(soldCount)
    
    // WRONG - Complex transformation logic
    response := &pb.EventResponse{
        Id:               event.Id,
        Name:             event.Name,
        AvailableTickets: int32(availableTickets),
        Status:           event.Status,
    }
    
    return response, nil
}
```

**Why this is wrong**:
- Business logic can't be tested without gRPC
- Can't reuse logic in other handlers/CLI
- Violates Single Responsibility Principle
- Hard to maintain and evolve

### Correct (Best Practice)

```go
// ✅ GOOD - Thin handler, delegates to usecase
func (s *EventGRPC) GetEvent(ctx context.Context, req *pb.GetEventRequest) (*pb.EventResponse, error) {
    // Only input validation
    if req.EventId == "" {
        return nil, status.Error(codes.InvalidArgument, "event_id required")
    }
    
    // Delegate to usecase
    event, err := s.eventUsecase.GetEvent(ctx, req.EventId)
    if err != nil {
        return nil, toGRPCError(err)  // Convert domain error to gRPC
    }
    
    // Only proto conversion
    return toProtoEvent(event), nil
}

// Helper: Convert domain error to gRPC status
func toGRPCError(err error) error {
    if errors.Is(err, domain.ErrNotFound) {
        return status.Error(codes.NotFound, "event not found")
    }
    if errors.Is(err, domain.ErrUnauthorized) {
        return status.Error(codes.PermissionDenied, "unauthorized")
    }
    return status.Error(codes.Internal, "internal error")
}

// Helper: Convert domain to proto (pure transformation)
func toProtoEvent(e *domain.Event) *pb.EventResponse {
    return &pb.EventResponse{
        Id:               e.Id,
        Name:             e.Name,
        Status:           string(e.Status),
        AvailableTickets: int32(e.AvailableTickets),
        // ... other fields
    }
}
```

**Why this is better**:
- Handler is thin (< 20 lines)
- Business logic in usecase (testable)
- Can reuse usecase in CLI/cron/etc
- Clear separation of concerns

### HTTP Handler Example

```go
// ❌ BAD - Business logic in HTTP handler
func (h *Handler) CreateOrder(c echo.Context) error {
    var req CreateOrderRequest
    if err := c.Bind(&req); err != nil {
        return c.JSON(400, "invalid request")
    }
    
    // WRONG - Fetching data in handler
    var event domain.Event
    h.db.First(&event, req.EventId)
    
    // WRONG - Business validation
    if event.Status != "active" {
        return c.JSON(400, "event not active")
    }
    
    // WRONG - Calculation
    totalPrice := req.Quantity * event.TicketPrice
    if req.PromoCode != "" {
        discount := h.calculateDiscount(req.PromoCode, totalPrice)
        totalPrice -= discount
    }
    
    // WRONG - Creating order directly
    order := &domain.Order{
        EventId:    req.EventId,
        Quantity:   req.Quantity,
        TotalPrice: totalPrice,
    }
    h.db.Create(order)
    
    return c.JSON(200, order)
}

// ✅ GOOD - Thin handler
func (h *Handler) CreateOrder(c echo.Context) error {
    var req CreateOrderRequest
    if err := c.Bind(&req); err != nil {
        return c.JSON(400, ErrorResponse{Message: "invalid request"})
    }
    
    // Basic validation only
    if req.EventId == "" || req.Quantity <= 0 {
        return c.JSON(400, ErrorResponse{Message: "invalid input"})
    }
    
    // Delegate to usecase
    order, err := h.orderUsecase.CreateOrder(c.Request().Context(), req.EventId, req.Quantity, req.PromoCode)
    if err != nil {
        return handleError(c, err)
    }
    
    return c.JSON(200, toOrderResponse(order))
}
```

### What Belongs in Delivery Layer

**✅ Allowed** (Protocol Concerns):
- Input validation (required fields, format)
- Request parsing/binding
- Response marshaling
- HTTP status code mapping
- gRPC status code mapping
- Authentication/authorization checks
- Rate limiting
- Request logging

**❌ Not Allowed** (Business Concerns):
- Database queries
- Business rule validation
- Calculations
- Data transformations
- External API calls
- Complex workflows
- Transaction management

### Pattern: Thin Handler Template

```go
// ✅ Standard handler pattern (5 steps)
func (h *Handler) HandleRequest(c echo.Context) error {
    // 1. Parse request
    var req RequestDTO
    if err := c.Bind(&req); err != nil {
        return c.JSON(400, "invalid request")
    }
    
    // 2. Basic validation
    if req.ID == "" {
        return c.JSON(400, "id required")
    }
    
    // 3. Delegate to usecase
    result, err := h.usecase.Execute(c.Request().Context(), req.ID)
    if err != nil {
        return handleError(c, err)
    }
    
    // 4. Convert to response DTO
    response := toResponseDTO(result)
    
    // 5. Return
    return c.JSON(200, response)
}
```

### Real-World Example from Your Codebase

```go
// ❌ Potential issue - If handler has this:
func (s *EventGRPC) SubmitEvent(ctx context.Context, req *pb.SubmitEventRequest) (*pb.EventResponse, error) {
    // WRONG - Complex validation logic
    event, _ := s.eventRepo.FindById(req.EventId)
    
    // WRONG - Business rules
    if event.Status == "draft" {
        // Count assets
        assets, _ := s.assetRepo.FindByEventId(event.Id)
        if len(assets) < 1 {
            return nil, status.Error(codes.FailedPrecondition, "need at least 1 asset")
        }
        
        // Count tickets
        tickets, _ := s.ticketRepo.FindByEventId(event.Id)
        if len(tickets) < 1 {
            return nil, status.Error(codes.FailedPrecondition, "need at least 1 ticket")
        }
        
        // Send emails
        s.emailClient.SendSubmissionEmail(...)
        
        // Update status
        event.Status = "on_review"
        s.eventRepo.Update(event)
    }
    
    return toProto(event), nil
}

// ✅ GOOD - Thin handler
func (s *EventGRPC) SubmitEvent(ctx context.Context, req *pb.SubmitEventRequest) (*pb.EventResponse, error) {
    if req.EventId == "" {
        return nil, status.Error(codes.InvalidArgument, "event_id required")
    }
    
    // All business logic in usecase
    event, err := s.eventUsecase.SubmitEvent(ctx, req.EventId)
    if err != nil {
        return nil, toGRPCError(err)
    }
    
    return toProtoEvent(event), nil
}
```

### Testing Benefits

```go
// ❌ BAD - Can't test business logic without gRPC
func TestGetEvent(t *testing.T) {
    // Need to set up entire gRPC server!
    server := setupGRPCServer()
    // ...
}

// ✅ GOOD - Test usecase directly
func TestGetEventUsecase(t *testing.T) {
    // Simple unit test
    mockRepo := &MockEventRepo{}
    usecase := NewEventUsecase(mockRepo)
    
    event, err := usecase.GetEvent(ctx, "123")
    assert.NoError(t, err)
    assert.Equal(t, "Event Name", event.Name)
}
```

### Quick Decision Tree

**Am I in a handler function?**
- Yes → **Is this protocol concern?**
  - Input validation → **OK**
  - DB query → **Move to usecase**
  - Business rule → **Move to usecase**
  - Calculation → **Move to usecase**

**Checklist for handlers**:
- [ ] < 30 lines per handler
- [ ] No database queries
- [ ] No business logic
- [ ] Only calls 1-2 usecase methods
- [ ] Pure transformation to/from proto/JSON

### References

- "Clean Architecture" by Robert C. Martin - Chapter 22
- [Hexagonal Architecture](https://alistair.cockburn.us/hexagonal-architecture/)
- [Go Clean Architecture Guide](https://threedots.tech/post/ddd-lite-in-go-introduction/)

---

**Rule Version**: 1.0  
**Last Updated**: 2026-01-21  
**Priority**: HIGH  
**Auto-fixable**: No (requires refactoring)
