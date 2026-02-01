# Assertive Programming

> "If It Can't Happen, Use Assertions to Ensure That It Won't"
> — David Thomas & Andrew Hunt

## Definition

**Assertive Programming** is the practice of using assertions to validate assumptions about your code's state during execution. Assertions are executable documentation that states what you believe to be true at specific points in your program. If an assertion fails, it indicates a programming error—a violation of your code's fundamental assumptions.

Think of assertions as tripwires for impossible conditions. They catch bugs early, document invariants, and serve as executable specifications of your code's contracts.

## Core Principle

**If something "can't happen," write an assertion to guarantee it won't.**

Developers often make implicit assumptions:
- "This parameter will never be null"
- "This array will always have at least one element"
- "This calculation will never produce a negative value"

Assertive programming makes these assumptions **explicit and verifiable**.

```pseudocode
// Implicit assumption: discountRate is between 0 and 1
function calculateDiscount(price, discountRate)
    return price * discountRate

// Explicit assertion: discountRate must be valid
function calculateDiscountSafe(price, discountRate)
    assert(discountRate >= 0 AND discountRate <= 1,
           "Discount rate must be between 0 and 1")
    return price * discountRate
```

## Assertions vs Error Handling

| Aspect | Assertions | Error Handling |
|--------|-----------|----------------|
| **Purpose** | Catch programmer errors | Handle runtime conditions |
| **When to use** | "This should never happen" | "This might happen" |
| **Example** | Internal invariant violation | User enters invalid input |
| **Recovery** | Program should crash/halt | Graceful recovery possible |
| **Audience** | Developers | End users |

### Examples

```pseudocode
// ASSERTION: Internal contract violation
function divide(numerator, denominator)
    // Caller should have validated; this is a programming error
    assert(denominator != 0, "Denominator cannot be zero")
    return numerator / denominator

// ERROR HANDLING: Expected runtime condition
function getUserInput()
    input = readFromUser()
    if input.isEmpty()
        throw ValidationError("Input cannot be empty")
    return input
```

## What to Assert

### 1. Preconditions

Conditions that must be true before a routine executes.

```pseudocode
function withdrawMoney(account, amount)
    assert(account != null, "Account must not be null")
    assert(amount > 0, "Withdrawal amount must be positive")
    assert(account.balance >= amount, "Insufficient funds")

    account.balance = account.balance - amount
```

### 2. Postconditions

Conditions guaranteed to be true after a routine completes.

```pseudocode
function sortArray(array)
    originalLength = array.length

    // ... sorting logic ...

    assert(array.length == originalLength,
           "Array length changed during sort")
    assert(isSorted(array), "Array is not sorted")

    return array
```

### 3. Invariants

Conditions that must always hold true for a data structure.

```pseudocode
class BoundedQueue
    maxSize = 100
    items = []

    function enqueue(item)
        assert(items.length < maxSize,
               "Queue invariant violated: exceeds max size")
        items.add(item)
        assert(items.length <= maxSize, "Post-condition failed")

    function dequeue()
        assert(items.length > 0, "Cannot dequeue from empty queue")
        return items.removeFirst()
```

### 4. Unreachable Code Paths

Code that should never execute.

```pseudocode
function handleStatus(status)
    switch status
        case SUCCESS:
            return processSuccess()
        case ERROR:
            return processError()
        case PENDING:
            return processPending()
        default:
            assert(false, "Unknown status: " + status)
```

## Leave Assertions On (The Pragmatic View)

### Traditional View

Many programming communities recommend disabling assertions in production for performance reasons.

### Pragmatic View

**Leave assertions enabled in production.**

**Why?**

1. **Bugs don't disappear in production**
   - The conditions you're asserting against are bugs
   - Production has unique data, load, and timing that testing can't replicate

2. **Fail fast is better than corrupt slowly**
   - Immediate crash is preferable to silent data corruption
   - Assertions catch bugs at the source, not after cascading failures

3. **Performance cost is negligible**
   - Modern compilers optimize assertions efficiently
   - Most assertions are simple boolean checks
   - The cost of a bug is far higher than CPU cycles

4. **Production-only bugs exist**
   - Edge cases appear under real load
   - Assertions are your last line of defense

```pseudocode
// Without assertions in production:
function transfer(fromAccount, toAccount, amount)
    fromAccount.balance = fromAccount.balance - amount
    toAccount.balance = toAccount.balance + amount
    // Bug: negative balance undetected, corrupts financial data

// With assertions enabled:
function transfer(fromAccount, toAccount, amount)
    assert(amount > 0, "Transfer amount must be positive")
    assert(fromAccount.balance >= amount, "Insufficient funds")

    fromAccount.balance = fromAccount.balance - amount
    toAccount.balance = toAccount.balance + amount

    assert(fromAccount.balance >= 0, "Balance went negative!")
    // Catches the bug immediately, prevents data corruption
```

## When NOT to Use Assertions

| Scenario | Use Instead |
|----------|-------------|
| Validating user input | Input validation + error handling |
| Checking external system responses | Error handling with retry logic |
| Network/IO failures | Exception handling |
| Business logic validation | Explicit conditional checks |
| Security checks | Never rely on assertions alone |

```pseudocode
// WRONG: Don't use assertions for security
function authenticateUser(username, password)
    assert(password.length > 8, "Password too short")
    // Attacker could disable assertions!

// RIGHT: Explicit security validation
function authenticateUser(username, password)
    if password.length < 8
        throw SecurityError("Password must be at least 8 characters")
```

## Best Practices

1. **Make assertion messages descriptive**
   ```pseudocode
   // Bad
   assert(x > 0)

   // Good
   assert(x > 0, "Product quantity must be positive, got: " + x)
   ```

2. **Assert the impossible, handle the unlikely**
   ```pseudocode
   // Impossible (programming error)
   assert(pointer != null, "Null pointer in initialized object")

   // Unlikely but possible (runtime condition)
   if fileExists(path) == false
       throw FileNotFoundError("File does not exist: " + path)
   ```

3. **Don't put side effects in assertions**
   ```pseudocode
   // WRONG
   assert(list.remove(item) == true)

   // RIGHT
   wasRemoved = list.remove(item)
   assert(wasRemoved == true, "Item not found in list")
   ```

## Summary Table

| Principle | Implementation |
|-----------|----------------|
| **Purpose** | Validate assumptions and catch programmer errors |
| **Placement** | Preconditions, postconditions, invariants, unreachable code |
| **Message quality** | Descriptive, includes context and actual values |
| **Production use** | Leave enabled (pragmatic approach) |
| **Not for** | User input, external systems, business logic, security |
| **Benefit** | Fail fast, self-documenting code, early bug detection |

**Key Takeaway**: Assertions are executable documentation of your code's contracts. They transform implicit assumptions into explicit, verifiable statements.

---

*Based on concepts from "The Pragmatic Programmer" by David Thomas and Andrew Hunt.*
