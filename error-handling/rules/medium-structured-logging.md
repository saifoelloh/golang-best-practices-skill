---
title: Log Database Errors with Structured Fields
impact: MEDIUM
impactDescription: Unstructured error logs can't be queried in APM/log aggregation tools
tags: logging, structured-logging, zerolog, zap, slog, observability
source: Uber Go Style Guide - Logging
---

## Log Database Errors with Structured Fields

Unstructured `log.Printf("error: %v", err)` produces logs that cannot be filtered, aggregated, or alerted on in tools like Loki, Datadog, or CloudWatch. Structured logging with key-value fields enables proper observability.

**Impact**: MEDIUM - Logs exist but are unsearchable; harder incident response

### Incorrect (Anti-Pattern)

```go
// ❌ BAD - unstructured, unsearchable
log.Printf("error finding user %d: %v", userID, err)
```

### Correct (Best Practice)

```go
// ✅ GOOD - zerolog (or zap / slog equivalent)
logger.Error().
    Err(err).
    Uint64("user_id", userID).
    Str("operation", "UserRepository.FindByID").
    Msg("database query failed")

// ✅ GOOD - slog (stdlib Go 1.21+)
slog.ErrorContext(ctx, "database query failed",
    "error", err,
    "user_id", userID,
    "operation", "UserRepository.FindByID",
)
```

### References

- [Uber Go Style Guide: Logging](https://github.com/uber-go/guide/blob/master/style.md#logging)
- [Go slog package](https://pkg.go.dev/log/slog)

---

**Rule Version**: 1.0
**Last Updated**: 2026-03-13
**Priority**: MEDIUM
**Auto-fixable**: No
