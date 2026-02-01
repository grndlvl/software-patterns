# Code Smells

## Overview

> "A code smell is a surface indication that usually corresponds to a deeper problem in the system."
> — Martin Fowler

Code smells are patterns that indicate potential problems. They're not bugs—the code works—but they suggest the design could be improved.

## Categories of Smells

### Comments

| Smell | Description | Refactoring |
|-------|-------------|-------------|
| **Inappropriate Information** | Comments with metadata (author, date, changelog) | Use version control |
| **Obsolete Comment** | Old comment that no longer applies | Delete or update |
| **Redundant Comment** | Comment that says what code says | Delete |
| **Poorly Written Comment** | Unclear, grammatically incorrect | Rewrite clearly |
| **Commented-Out Code** | Dead code in comments | Delete (use VCS) |

### Environment

| Smell | Description | Refactoring |
|-------|-------------|-------------|
| **Build Requires Multiple Steps** | Complex build process | Automate with single command |
| **Tests Require Multiple Steps** | Complex test setup | Automate with single command |

### Functions

| Smell | Description | Refactoring |
|-------|-------------|-------------|
| **Too Many Arguments** | More than 3 parameters | Introduce Parameter Object |
| **Output Arguments** | Function modifies arguments | Return value instead |
| **Flag Arguments** | Boolean changes behavior | Split into two functions |
| **Dead Function** | Function never called | Delete |

### General

| Smell | Description | Refactoring |
|-------|-------------|-------------|
| **Multiple Languages in One File** | HTML/CSS/JS mixed | Separate files |
| **Obvious Behavior Not Implemented** | Function doesn't do what name suggests | Implement or rename |
| **Incorrect Behavior at Boundaries** | Edge cases not handled | Add boundary tests |
| **Duplication** | Same code in multiple places | Extract Method/Class |
| **Code at Wrong Level of Abstraction** | Mixed high/low level | Separate abstractions |
| **Base Classes Depending on Derivatives** | Parent knows child details | Invert dependency |
| **Too Much Information** | Class exposes too much | Hide implementation |
| **Dead Code** | Unreachable code | Delete |
| **Vertical Separation** | Related code far apart | Move together |
| **Inconsistency** | Similar things done differently | Standardize |
| **Clutter** | Unused variables, imports | Delete |
| **Artificial Coupling** | Unrelated things bundled | Separate |
| **Feature Envy** | Method uses other class more than own | Move method |
| **Selector Arguments** | Argument selects behavior | Polymorphism |
| **Obscured Intent** | Hard to understand code | Clarify naming |
| **Misplaced Responsibility** | Code in wrong class | Move to right class |
| **Inappropriate Static** | Static when instance makes sense | Make instance |
| **Use Explanatory Variables** | Complex expressions | Extract to named variable |
| **Function Names Should Say What They Do** | Unclear names | Rename |
| **Understand the Algorithm** | Code works by accident | Understand and clarify |
| **Make Logical Dependencies Physical** | Implicit dependencies | Make explicit |
| **Prefer Polymorphism to If/Else or Switch** | Type-based conditionals | Polymorphism |
| **Follow Standard Conventions** | Breaking team standards | Conform |
| **Replace Magic Numbers with Named Constants** | Literal numbers | Extract constant |
| **Be Precise** | Ambiguous code | Be specific |
| **Structure Over Convention** | Relying on naming only | Use types/structure |
| **Encapsulate Conditionals** | Complex boolean logic | Extract to named function |
| **Avoid Negative Conditionals** | `if (!isNotFound)` | Use positive form |
| **Functions Should Do One Thing** | Multiple responsibilities | Extract functions |
| **Hidden Temporal Couplings** | Order dependencies implicit | Make explicit |
| **Don't Be Arbitrary** | Unexplained structure | Communicate intent |
| **Encapsulate Boundary Conditions** | Repeated boundary logic | Extract method |
| **Functions Should Descend One Level of Abstraction** | Mixed levels | Consistent abstraction |
| **Keep Configurable Data at High Levels** | Config buried in code | Centralize config |
| **Avoid Transitive Navigation** | a.getB().getC().doThing() | Law of Demeter |

### Names

| Smell | Description | Refactoring |
|-------|-------------|-------------|
| **Choose Descriptive Names** | `d` instead of `elapsedDays` | Rename |
| **Choose Names at Appropriate Abstraction Level** | Implementation details in name | Abstract the name |
| **Use Standard Nomenclature Where Possible** | Custom terms for known patterns | Use pattern names |
| **Unambiguous Names** | Multiple interpretations possible | Clarify |
| **Use Long Names for Long Scopes** | Short names for long-lived variables | Lengthen |
| **Avoid Encodings** | Hungarian notation | Remove prefixes |
| **Names Should Describe Side Effects** | `createUser()` also sends email | Rename or split |

### Tests

| Smell | Description | Refactoring |
|-------|-------------|-------------|
| **Insufficient Tests** | Missing edge cases | Add tests |
| **Use a Coverage Tool** | Unknown coverage | Measure and improve |
| **Don't Skip Trivial Tests** | "Too simple to test" | Test anyway |
| **Ignored Test is a Question** | Skipped/disabled tests | Fix or delete |
| **Test Boundary Conditions** | Only happy path tested | Add boundary tests |
| **Exhaustively Test Near Bugs** | Bug found, minimal fix | Test thoroughly around it |
| **Patterns of Failure Are Revealing** | Similar tests fail | Investigate pattern |
| **Test Coverage Patterns Can Be Revealing** | Uncovered code | Add coverage |
| **Tests Should Be Fast** | Slow tests | Optimize or mock |

## Detailed Examples

### Duplication (DRY Violation)

```pseudocode
// Bad - duplicated validation
function createUser(data) {
    if data.email == null || data.email == "" {
        throw ValidationError("Email required")
    }
    if not data.email.contains("@") {
        throw ValidationError("Invalid email format")
    }
    // ... create user
}

function updateUser(data) {
    if data.email == null || data.email == "" {
        throw ValidationError("Email required")
    }
    if not data.email.contains("@") {
        throw ValidationError("Invalid email format")
    }
    // ... update user
}

// Good - extracted
function validateEmail(email) {
    if email == null || email == "" {
        throw ValidationError("Email required")
    }
    if not email.contains("@") {
        throw ValidationError("Invalid email format")
    }
}

function createUser(data) {
    validateEmail(data.email)
    // ... create user
}

function updateUser(data) {
    validateEmail(data.email)
    // ... update user
}
```

### Feature Envy

```pseudocode
// Bad - method uses other class's data extensively
class Order {
    customer

    function getShippingAddress() {
        // Envies Customer's data
        return customer.street + "\n" +
               customer.city + ", " +
               customer.state + " " +
               customer.zipCode
    }
}

// Good - move to the class that owns the data
class Customer {
    street
    city
    state
    zipCode

    function getFormattedAddress() {
        return street + "\n" +
               city + ", " +
               state + " " +
               zipCode
    }
}

class Order {
    customer

    function getShippingAddress() {
        return customer.getFormattedAddress()
    }
}
```

### Long Method

```pseudocode
// Bad - method too long
function processOrder(order) {
    // Validate (20 lines)
    // Calculate totals (15 lines)
    // Apply discounts (25 lines)
    // Check inventory (20 lines)
    // Reserve inventory (15 lines)
    // Process payment (30 lines)
    // Create shipment (20 lines)
    // Send notifications (25 lines)
    // Log everything (15 lines)
    // Total: 185 lines!
}

// Good - extracted methods
function processOrder(order) {
    validateOrder(order)
    calculateTotals(order)
    applyDiscounts(order)
    reserveInventory(order)
    processPayment(order)
    createShipment(order)
    sendNotifications(order)
    logOrderProcessed(order)
}
```

### Primitive Obsession

```pseudocode
// Bad - using primitives for domain concepts
function createUser(
    name: string,
    email: string,
    phone: string,
    streetAddress: string,
    city: string,
    state: string,
    zipCode: string
) { ... }

// Good - value objects
class Email {
    value

    constructor(email: string) {
        if not isValidEmail(email) {
            throw InvalidEmailError
        }
        value = email
    }
}

class Address {
    street
    city
    state
    zipCode

    function format() { ... }
}

function createUser(name: string, email: Email, phone: Phone, address: Address) {
    ...
}
```

### Switch Statements

```pseudocode
// Bad - switch on type
function calculatePay(employee) {
    switch employee.type {
        case COMMISSIONED:
            return calculateCommissionedPay(employee)
        case HOURLY:
            return calculateHourlyPay(employee)
        case SALARIED:
            return calculateSalariedPay(employee)
    }
}

// Good - polymorphism
interface Employee {
    function calculatePay()
}

class CommissionedEmployee implements Employee {
    function calculatePay() {
        return basePay + commission
    }
}

class HourlyEmployee implements Employee {
    function calculatePay() {
        return hoursWorked * hourlyRate
    }
}

class SalariedEmployee implements Employee {
    function calculatePay() {
        return monthlySalary
    }
}
```

### Data Clumps

```pseudocode
// Bad - same parameters always together
function createRectangle(x, y, width, height) { ... }
function moveRectangle(x, y, width, height, newX, newY) { ... }
function resizeRectangle(x, y, width, height, newWidth, newHeight) { ... }

// Good - extract class
class Rectangle {
    x, y, width, height

    function move(newX, newY) { ... }
    function resize(newWidth, newHeight) { ... }
}
```

## Refactoring Catalog

| Smell | Common Refactoring |
|-------|-------------------|
| Duplicated Code | Extract Method, Pull Up Method |
| Long Method | Extract Method, Replace Temp with Query |
| Large Class | Extract Class, Extract Subclass |
| Long Parameter List | Introduce Parameter Object |
| Divergent Change | Extract Class |
| Shotgun Surgery | Move Method, Move Field |
| Feature Envy | Move Method |
| Data Clumps | Extract Class |
| Primitive Obsession | Replace Primitive with Object |
| Switch Statements | Replace with Polymorphism |
| Parallel Inheritance | Move Method, Move Field |
| Lazy Class | Inline Class |
| Speculative Generality | Collapse Hierarchy |
| Temporary Field | Extract Class |
| Message Chains | Hide Delegate |
| Middle Man | Remove Middle Man |
| Inappropriate Intimacy | Move Method, Extract Class |
| Comments | Extract Method, Rename |

---

*Based on concepts from "Clean Code" by Robert C. Martin and "Refactoring" by Martin Fowler.*
