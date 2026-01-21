# Design Pattern & Refactoring Rules - Implementation Plan

**Created:** 2026-01-22  
**Skill Version:** 1.1.0 (Expansion)  
**Goal:** Add practical design pattern & refactoring rules

---

## 🎯 Motivation

Based on today's real refactoring session, we identified common issues in AI-generated and production Go code:

1. ✅ **God functions** (500+ lines) - FIXED today with FeeCalculator
2. ✅ **Magic numbers** - FIXED today with constants
3. ⚠️ **Missing domain entities** - Partially addressed
4. ⚠️ **Feature envy** - Usecases accessing deep into other objects
5. ⚠️ **Primitive obsession** - Using primitives instead of value objects
6. ⚠️ **Long parameter lists** - 7+ parameters common
7. ⚠️ **Data clumps** - Same groups of parameters everywhere

**Key Insight:** These patterns are EXTREMELY common in AI-generated code and real codebases!

---

## 📊 Proposed Rules

### Category: Design Patterns & Refactoring
**Priority Mix:** 2 HIGH + 8 MEDIUM = 10 new rules

---

## 🔴 HIGH Priority Rules (2)

### 1. high-god-object

**Title:** Extract Domain Logic from God Objects  
**Impact:** Functions/structs with >300 lines or >10 responsibilities

**Detection:**
- Functions >200 lines (consider medium complexity)
- Functions >300 lines (definitely refactor)
- Structs with >15 methods doing unrelated things
- Single function handling 5+ distinct responsibilities

**Example:**
```go
// ❌ BAD - God function (500 lines)
func CreateTransactionByOrder(ctx, orderId, paymentMethodId, ...) {
    // Validate order (50 lines)
    // Calculate fees (195 lines)
    // Create payment (40 lines)
    // Process xendit (80 lines)
    // Send email (30 lines)
    // Update cache (50 lines)
    // Create tickets (100 lines)
}

// ✅ GOOD - Extracted responsibilities
func CreateTransactionByOrder(ctx, orderId, paymentMethodId, ...) {
    order := u.validateAndGetOrder(ctx, orderId)
    feeCalc := u.calculateFees(ctx, order, paymentMethodId)
    payment := u.createPayment(ctx, feeCalc)
    u.processPayment(ctx, payment)
    u.sendNotifications(ctx, order, payment)
    tickets := u.createTickets(ctx, order, payment)
    return NewTransaction(order, payment, tickets)
}
```

**Why HIGH:** Directly impacts maintainability, testability, and team velocity

**Source:** Clean Code (Robert C. Martin), Refactoring (Martin Fowler)

---

### 2. high-extract-method

**Title:** Extract Complex Logic to Named Methods  
**Impact:** Improves readability, naming documents intent

**Detection:**
- Code blocks with comments explaining what they do
- Nested blocks >3 levels deep
- Repeated logic patterns
- Complex conditionals that need comments

**Example:**
```go
// ❌ BAD - Complex inline logic
func ProcessWithdrawal(w *Withdrawal) error {
    // Calculate progressive tax on revenue
    totalRev := w.RequestedAmount
    taxRate := 0.11
    if *e.Tax <= 100 {
        taxPortion = totalRev * taxRate / (1.0 + taxRate)
    } else {
        taxPortion = int64(*e.Tax)
    }
    
    // Calculate tier-based fee
    if revenue < 100000000 {
        fee = revenue * 0.01
    } else if revenue < 500000000 {
        fee = revenue * 0.008
    }
    // ... 100 more lines
}

// ✅ GOOD - Extracted methods
func ProcessWithdrawal(w *Withdrawal) error {
    taxPortion := calculateTaxPortion(w, event)
    fee := calculateTierFee(revenue)
    amountReceived := w.RequestedAmount - fee - taxPortion
    return u.repo.Create(w)
}

func calculateTaxPortion(w *Withdrawal, e *Event) int64 {
    // Clear responsibility
}

func calculateTierFee(revenue int64) int64 {
    // Clear responsibility
}
```

**Why HIGH:** Fundamental refactoring that enables all other improvements

**Source:** Refactoring (Martin Fowler), Clean Code

---

## 🟡 MEDIUM Priority Rules (8)

### 3. medium-primitive-obsession

**Title:** Replace Primitives with Value Objects  
**Impact:** Type safety, validation, domain clarity

**Detection:**
- Email, phone, UUID as `string`
- Money amounts as `int64` or `float64`
- Status/enum as `string` instead of custom type
- Validation scattered instead of in constructor

**Example:**
```go
// ❌ BAD - Primitive obsession
type Event struct {
    Email       string // Any string, no validation
    PhoneNumber string // Format unknown
    Status      string // What are valid values?
}

func (e *Event) SetStatus(s string) {
    if s == "active" || s == "inactive" {
        e.Status = s
    }
}

// ✅ GOOD - Value objects
type Email struct {
    value string
}

func NewEmail(email string) (Email, error) {
    if !isValidEmail(email) {
        return Email{}, errors.New("invalid email")
    }
    return Email{value: email}, nil
}

type EventStatus string

const (
    EventStatusActive   EventStatus = "active"
    EventStatusInactive EventStatus = "inactive"
)

type Event struct {
    Email       Email
    PhoneNumber PhoneNumber
    Status      EventStatus
}
```

**Source:** DDD (Eric Evans), Refactoring

---

### 4. medium-long-parameter-list

**Title:** Replace Long Parameter Lists with Structs  
**Impact:** Readability, flexibility, IDE support

**Detection:**
- Functions with >5 parameters
- Same parameter groups repeated
- Optional parameters using pointers

**Example:**
```go
// ❌ BAD - 8 parameters
func CreateEvent(name, desc, location, startDate, endDate string, 
                 price int64, tax *int64, adminFee *int64) error

// ✅ GOOD - Parameter object
type CreateEventParams struct {
    Name       string
    Desc       string
    Location   string
    StartDate  string
    EndDate    string
    Price      int64
    Tax        *int64
    AdminFee   *int64
}

func CreateEvent(params CreateEventParams) error
```

**Source:** Refactoring (Martin Fowler)

---

### 5. medium-data-clumps

**Title:** Extract Repeated Parameter Groups  
**Impact:** DRY, clear domain concepts

**Detection:**
- Same 3+ parameters appear together in 3+ functions
- Parameters always passed as a group
- Missing cohesive domain concept

**Example:**
```go
// ❌ BAD - Data clump (eventId, eventOrganizerId, userId everywhere)
func GetTickets(eventId, eventOrganizerId, userId string) ([]*Ticket, error)
func GetOrders(eventId, eventOrganizerId, userId string) ([]*Order, error)
func GetTransactions(event Id, eventOrganizerId, userId string) ([]*Tx, error)

// ✅ GOOD - Extract to struct
type EventContext struct {
    EventID          string
    EventOrganizerID string
    UserID           string
}

func GetTickets(ctx EventContext) ([]*Ticket, error)
func GetOrders(ctx EventContext) ([]*Order, error)
func GetTransactions(ctx EventContext) ([]*Tx, error)
```

**Source:** Refactoring (Martin Fowler)

---

### 6. medium-feature-envy

**Title:** Move Logic Closer to Data  
**Impact:** Encapsulation, cohesion

**Detection:**
- Method accesses another object's fields >3 times
- Complex calculations on external data
- Business logic in wrong layer

**Example:**
```go
// ❌ BAD - Usecase envies PartnerTicketType's data
func (u *OrderUsecase) CalculateDiscount(ptt *PartnerTicketType, price int64) int64 {
    discount := ptt.GetDiscount(ticketTypeId)
    if discount.Type == "percentage" {
        return price * discount.Value / 100
    }
    return discount.Value
}

// ✅ GOOD - Move to domain
func (ptt *PartnerTicketType) CalculateDiscountedPrice(price int64, ticketTypeId string) int64 {
    discount := ptt.GetDiscount(ticketTypeId)
    if discount.Type == "percentage" {
        return price * discount.Value / 100
    }
    return discount.Value
}
```

**Source:** Refactoring (Martin Fowler)

---

### 7. medium-magic-constants

**Title:** Replace Magic Numbers/Strings with Constants  
**Impact:** Readability, maintainability

**Detection:**
- Literals used directly in code (except 0, 1, -1)
- Same number appears in multiple places
- No clear meaning from context

**Example:**
```go
// ❌ BAD - What is 2000? What is 100?
withdrawalFee := int64(2000)
if tax <= 100 {
    // percentage
}

// ✅ GOOD - Named constants
const (
    DefaultWithdrawalFee    = 2000
    TaxPercentageThreshold  = 100
)

withdrawalFee := DefaultWithdrawalFee
if tax <= TaxPercentageThreshold {
    // percentage - clear!
}
```

**Source:** Clean Code (Robert C. Martin)

---

### 8. medium-builder-pattern

**Title:** Use Builder for Complex Object Construction  
**Impact:** Flexibility, readability

**Detection:**
- Constructors with >5 parameters
- Many optional fields
- Complex initialization logic

**Example:**
```go
// ❌ BAD - Hard to use
fee := NewFeeCalculator(event, ticket, payment, order, groupTicket, partner)

// ✅ GOOD - Builder pattern
calculator := NewFeeCalculator(event, ticket, payment, order).
    WithGroupTicket(groupTicket).
    WithPartnerDiscount(partner).
    Calculate()
```

**Source:** Effective Go, Design Patterns (Gang of Four)

---

### 9. medium-factory-constructor

**Title:** Use Factory Functions for Complex Creation  
**Impact:** Encapsulation, validation

**Detection:**
- Constructors with complex logic
- Multiple ways to create same object
- Creation requires validation

**Example:**
```go
// ❌ BAD - Manual construction
event := &Event{
    Id: uuid.New().String(),
    Status: "draft",
    CreatedAt: time.Now(),
}

// ✅ GOOD - Factory function
func NewDraftEvent(name string, organizerId string) (*Event, error) {
    if name == "" {
        return nil, errors.New("name required")
    }
    return &Event{
        Id:          uuid.New().String(),
        Name:        name,
        OrganizerId: organizerId,
        Status:      EventStatusDraft,
        CreatedAt:   time.Now(),
    }, nil
}
```

**Source:** Effective Go, Clean Architecture

---

### 10. medium-introduce-parameter-object

**Title:** Group Related Parameters into Struct  
**Impact:** Same as long-parameter-list but with emphasis on domain modeling

**Example:**
```go
// ❌ BAD
func CreateFee(basePrice, adminFee, taxFee, paymentFee int64) *Fee

// ✅ GOOD
type FeeComponents struct {
    BasePrice     int64
    AdminFee      int64
    TaxFee        int64
    PaymentFee    int64
}

func CreateFee(components FeeComponents) *Fee
```

---

## 📋 Implementation Plan

### Phase 1: Create Rule Files (3 hours)

**HIGH Priority:**
1. `rules/high-god-object.md`
2. `rules/high-extract-method.md`

**MEDIUM Priority:**
3. `rules/medium-primitive-obsession.md`
4. `rules/medium-long-parameter-list.md`
5. `rules/medium-data-clumps.md`
6. `rules/medium-feature-envy.md`
7. `rules/medium-magic-constants.md`
8. `rules/medium-builder-pattern.md`
9. `rules/medium-factory-constructor.md`
10. `rules/medium-introduce-parameter-object.md`

### Phase 2: Update Metadata (30 min)

**Update files:**
- `rules/_categories.md` - Add new section
- `SKILL.md` - Update rule count (30 → 40)
- `README.md` - Update statistics
- `metadata.json` - Bump version to 1.1.0

### Phase 3: Real-World Examples (1 hour)

Use examples from TODAY's refactoring:
- FeeCalculator extraction (god-object, extract-method)
- Constants creation (magic-constants)
- Builder pattern usage (builder-pattern)

### Phase 4:Update Quick Reference (30 min)

Add new rules to SKILL.md quick reference section

---

## 🎯 Success Metrics

**Coverage:**
- Total rules: 30 → 40 (+33%)
- Design patterns: 0 → 10 (NEW category)
- Real examples: From production code

**Impact:**
- Catches AI-generated anti-patterns
- Guides refactoring decisions
- Complements existing architecture rules

---

## 💡 Why These 10 Rules?

**Based on Real Pain Points:**
1. ✅ We literally fixed `god-object` today (CreateTransactionByOrder)
2. ✅ We used `extract-method` pattern (FeeCalculator)
3. ✅ We eliminated `magic-constants` today
4. ✅ We used `builder-pattern` (WithGroupTicket)
5. ⚠️ We've seen `primitive-obsession` (string status everywhere)
6. ⚠️ We've seen `long-parameter-list` (8+ params common)
7. ⚠️ We've seen `feature-envy` (usecase reaching into domains)
8. ⚠️ We've seen `data-clumps` (eventId, organizerId, userId)

**These are PROVEN to exist and PROVEN to be fixable!**

---

## 📚 Reference Sources

All rules backed by authoritative sources:
- ✅ Refactoring (Martin Fowler)
- ✅ Clean Code (Robert C. Martin)
- ✅ Design Patterns (Gang of Four)
- ✅ Domain-Driven Design (Eric Evans)
- ✅ Effective Go (Go team)

---

## 🚀 Version Roadmap

**v1.0.0** (Current)
- 30 rules (8 CRITICAL, 12 HIGH, 10 MEDIUM)
- Focus: Correctness, Architecture, Concurrency

**v1.1.0** (Proposed)
- 40 rules (8 CRITICAL, 14 HIGH, 18 MEDIUM)
- Focus: + Design Patterns & Refactoring

**v1.2.0** (Future)
- Testing patterns
- Performance patterns
- Security patterns

---

##  🎯 Recommendation

**PROCEED** with all 10 rules!

**Why:**
1. Based on REAL refactoring experience (today!)
2. Extremely common in AI-generated code
3. Well-documented patterns with clear fixes
4. Complements existing skill perfectly
5. Immediate value for users

**Priority Order:**
1. Start with `high-god-object` and `high-extract-method` (biggest impact)
2. Then `medium-magic-constants` (quick win, we just did it)
3. Then others in any order

---

**Mau gue start implement sekarang?** 🚀
