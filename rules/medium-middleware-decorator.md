---
title: Use Decorator Pattern for Middleware
impact: MEDIUM
impactDescription: Composability, separation of concerns
tags: design-patterns, decorator, middleware, http
source: Refactoring.Guru; Design Patterns (GoF)
---

## Use Decorator Pattern for Middleware

Decorator pattern allows adding responsibilities dynamically. In Go, extremely common for http.Handler middleware chains.

**Impact**: MEDIUM - Composable, reusable cross-cutting concerns

### Detection

When to use:
- Cross-cutting concerns (logging, auth, metrics)
- Wrapping behavior
- Multiple decorators stacked
- http.Handler middleware

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - Mixed concerns in handler
func HandleRequest(w http.ResponseWriter, r *http.Request) {
    // Logging
    log.Printf("Request: %s %s", r.Method, r.URL.Path)
    
    // Auth
    if !isAuthenticated(r) {
        http.Error(w, "Unauthorized", 401)
        return
    }
    
    // Business logic
    // ...
}
```

**Why this is wrong**:
- Mixed concerns
- Cannot reuse logging/auth
- Hard to test

### Correct (Best Practice)

```go
// ✅ GOOD - Decorator middleware
func LoggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        log.Printf("Request: %s %s", r.Method, r.URL.Path)
        next.ServeHTTP(w, r)
    })
}

func AuthMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        if !isAuthenticated(r) {
            http.Error(w, "Unauthorized", 401)
            return
        }
        next.ServeHTTP(w, r)
    })
}

// Compose
handler := LoggingMiddleware(AuthMiddleware(businessHandler))
```

**Why this is better**:
- Separation of concerns
- Reusable decorators
- Testable independently
- Composable

### References

- [Refactoring.Guru - Decorator Pattern](https://refactoring.guru/design-patterns/decorator)
- Design Patterns (Gang of Four)

---

**Rule Version**: 1.0  
**Last Updated**: 2026-01-22
