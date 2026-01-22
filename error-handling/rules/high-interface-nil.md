---
title: Check Interface Nil Correctly
impact: HIGH
impactDescription: Interface containing nil pointer != nil, causes unexpected behavior
tags: interface, nil, pointer, type-system, gotcha
source: Learning Go - Chapter 7 (Interfaces)
---

## Check Interface Nil Correctly

An interface containing a nil pointer is not nil itself. This causes `if err != nil` checks to unexpectedly pass.

**Impact**: HIGH - Error handling logic breaks

### The Problem

```go
// ❌ Typed nil != nil
var err error = (*MyError)(nil)  // Interface with nil pointer
if err != nil {  // TRUE! Interface is not nil
    // Executes unexpectedly
}
```

### Solution

```go
// ✅ Return nil, not typed nil
func doSomething() error {
    var err *MyError
    if someCondition {
        err = &MyError{}
    }
    
    if err != nil {
        return err
    }
    
    return nil  // Return nil, not err (typed nil)
}
```

---

**Rule Version**: 1.0  
**Priority**: HIGH
