# DRY - Don't Repeat Yourself

## Definition

> "Every piece of knowledge must have a single, unambiguous, authoritative representation within a system."
> — Dave Thomas & Andy Hunt

DRY is not just about code duplication—it's about knowledge duplication.

## What DRY Really Means

DRY applies to:
- **Code** - Obvious duplication
- **Documentation** - Comments that duplicate code
- **Data structures** - Redundant data storage
- **APIs** - Multiple representations of same concept
- **Developer knowledge** - Multiple developers solving same problem

## Types of Duplication

### Imposed Duplication

Duplication that seems required by the environment.

```pseudocode
// Problem: API schema duplicated in docs, code, and tests

// Solution: Generate from single source
// schema.yaml → API code
// schema.yaml → Documentation
// schema.yaml → Client SDKs
// schema.yaml → Test fixtures
```

### Inadvertent Duplication

Duplication developers don't realize they're creating.

```pseudocode
// Bad - duplicated business logic
class Order {
    function total() {
        return items.sum(i => i.price * i.quantity) * 1.1  // 10% tax
    }
}

class Invoice {
    function amount() {
        return lineItems.sum(l => l.unitPrice * l.qty) * 1.1  // 10% tax
    }
}

// Good - single source of truth
TAX_RATE = 1.1

class TaxCalculator {
    function applyTax(amount) {
        return amount * TAX_RATE
    }
}
```

### Impatient Duplication

Duplication from developers taking shortcuts.

```pseudocode
// Bad - "I'll just copy this function and modify it"
function validateUser(user) {
    if user.name.length < 2 { return false }
    if user.email.indexOf("@") < 0 { return false }
    return true
}

function validateAdmin(admin) {
    if admin.name.length < 2 { return false }
    if admin.email.indexOf("@") < 0 { return false }  // Duplicated!
    if admin.accessLevel < 5 { return false }
    return true
}

// Good - compose validators
function validatePerson(person) {
    return validateName(person.name) && validateEmail(person.email)
}

function validateUser(user) {
    return validatePerson(user)
}

function validateAdmin(admin) {
    return validatePerson(admin) && validateAccessLevel(admin.accessLevel)
}
```

### Inter-Developer Duplication

Different team members solving the same problem.

**Solutions:**
- Good communication
- Code reviews
- Shared libraries
- Documentation of existing solutions
- Regular team discussions

## Code Duplication

### The Rule of Three

1. First time: Just do it
2. Second time: Wince at duplication, but do it anyway
3. Third time: Refactor

### Extracting Duplication

```pseudocode
// Before - duplicated validation
function createUser(data) {
    if data.email == null || data.email == "" {
        throw Error("Email required")
    }
    if not data.email.match(EMAIL_REGEX) {
        throw Error("Invalid email")
    }
    // ... create user
}

function updateUser(id, data) {
    if data.email == null || data.email == "" {
        throw Error("Email required")
    }
    if not data.email.match(EMAIL_REGEX) {
        throw Error("Invalid email")
    }
    // ... update user
}

// After - single validation
function validateEmail(email) {
    if email == null || email == "" {
        throw Error("Email required")
    }
    if not email.match(EMAIL_REGEX) {
        throw Error("Invalid email")
    }
}

function createUser(data) {
    validateEmail(data.email)
    // ... create user
}

function updateUser(id, data) {
    validateEmail(data.email)
    // ... update user
}
```

## Documentation Duplication

### Bad: Comments That Duplicate Code

```pseudocode
// Bad - comment says what code says
// Add one to the counter
counter = counter + 1

// Good - comment explains why
// Compensate for zero-based index in display
counter = counter + 1
```

### Good: Generate Documentation from Code

```pseudocode
/**
 * Calculate compound interest.
 * @param principal - Initial amount
 * @param rate - Annual interest rate (0.05 = 5%)
 * @param years - Number of years
 * @returns Final amount after interest
 */
function calculateInterest(principal, rate, years) {
    return principal * (1 + rate) ** years
}

// Documentation generated from this single source
```

## Data Duplication

### Bad: Redundant Data

```pseudocode
// Bad - storing derived data
class Order {
    items = []
    itemCount = 0  // Redundant!
    subtotal = 0   // Redundant!
    total = 0      // Redundant!

    function addItem(item) {
        items.add(item)
        itemCount++  // Must keep in sync
        subtotal += item.price  // Must keep in sync
        total = subtotal * 1.1  // Must keep in sync
    }
}

// Good - calculate when needed
class Order {
    items = []

    function addItem(item) {
        items.add(item)
    }

    function itemCount() {
        return items.length
    }

    function subtotal() {
        return items.sum(i => i.price)
    }

    function total() {
        return subtotal() * 1.1
    }
}
```

### When Caching is OK

Caching derived data is acceptable when:
- Performance requires it
- The cache is clearly marked as cache
- There's a single mechanism to invalidate/update

```pseudocode
class Order {
    items = []
    _cachedTotal = null  // Clearly marked as cache

    function addItem(item) {
        items.add(item)
        _cachedTotal = null  // Invalidate cache
    }

    function total() {
        if _cachedTotal == null {
            _cachedTotal = calculateTotal()
        }
        return _cachedTotal
    }
}
```

## API/Schema Duplication

### Single Source of Truth

```yaml
# schema.yaml - THE source of truth
User:
  properties:
    id: integer
    name: string
    email: string
    createdAt: datetime
```

Generate everything from this:
- Database migrations
- API endpoints
- TypeScript interfaces
- Python dataclasses
- API documentation
- Client SDKs

## When Duplication is OK

### Coincidental Similarity

Two things that look the same but represent different concepts:

```pseudocode
// These look similar but serve different purposes
class UserValidator {
    function validate(user) {
        return user.name.length >= 2
    }
}

class ProductValidator {
    function validate(product) {
        return product.name.length >= 2  // Same logic, different domain
    }
}

// DON'T extract - they may evolve differently
// User names might need 2 chars, product names might need 5 later
```

### Performance-Critical Code

Sometimes inline duplication is faster than function calls.

## DRY vs. Premature Abstraction

Don't create abstractions too early:

```pseudocode
// Too early - only one use case
interface DataProcessor<T, R> {
    function process(input: T): R
}

class UserProcessor implements DataProcessor<RawUser, User> {
    // Lots of generic code for one use case
}

// Better - wait for patterns to emerge
class UserProcessor {
    function process(rawUser) {
        // Simple, direct implementation
    }
}

// Abstract when you see the third use case
```

## Summary

| Do | Don't |
|----|-------|
| Single source of truth | Copy-paste code |
| Generate from schemas | Maintain multiple copies |
| Calculate derived data | Store redundant data |
| Share common libraries | Solve same problem twice |
| Communicate with team | Assume others haven't solved it |

---

*Based on concepts from "The Pragmatic Programmer" by David Thomas and Andrew Hunt.*
