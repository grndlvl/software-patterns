# Formatting

## Core Principle

> "Code formatting is about communication, and communication is the professional developer's first order of business."
> — Robert C. Martin

Formatting is not about personal preference—it's about making code readable for the team.

## Vertical Formatting

### File Size

Keep files small and focused. Typical guidelines:
- Most files: 200-500 lines
- Upper limit: 500 lines (can be done well)
- Maximum: Rarely exceed 1000 lines

Smaller files are easier to understand.

### The Newspaper Metaphor

Code should read like a newspaper article:
- **Headline (name)**: Tells you what it's about
- **Synopsis (top)**: High-level concepts first
- **Details (bottom)**: Lower-level details below

```pseudocode
// File: UserRegistrationService

// High-level public API at top
class UserRegistrationService {

    function registerUser(userData) {
        validateUserData(userData)
        user = createUser(userData)
        sendWelcomeEmail(user)
        return user
    }

    // Supporting methods below
    private function validateUserData(data) {
        validateEmail(data.email)
        validatePassword(data.password)
        validateUsername(data.username)
    }

    private function validateEmail(email) { ... }
    private function validatePassword(password) { ... }
    private function validateUsername(username) { ... }

    // Lower-level details at bottom
    private function createUser(data) { ... }
    private function sendWelcomeEmail(user) { ... }
}
```

### Vertical Openness Between Concepts

Use blank lines to separate distinct thoughts:

```pseudocode
// Bad - cramped
function calculateTotal(items) {
    subtotal = 0
    for item in items {
        subtotal += item.price * item.quantity
    }
    tax = subtotal * TAX_RATE
    shipping = calculateShipping(items)
    total = subtotal + tax + shipping
    return total
}

// Good - concepts separated
function calculateTotal(items) {
    subtotal = 0
    for item in items {
        subtotal += item.price * item.quantity
    }

    tax = subtotal * TAX_RATE
    shipping = calculateShipping(items)

    total = subtotal + tax + shipping
    return total
}
```

### Vertical Density

Related code should be close together:

```pseudocode
// Bad - unnecessary separation
class ReportConfig {

    private reportName


    private reportType


    private outputFormat


    function getReportName() {
        return reportName
    }
}

// Good - related things together
class ReportConfig {
    private reportName
    private reportType
    private outputFormat

    function getReportName() {
        return reportName
    }
}
```

### Vertical Distance

Related concepts should be vertically close. Functions that call each other should be near each other.

```pseudocode
// Bad - caller far from callee
function high() {
    medium()
}

// ... 200 lines of other code ...

function medium() {
    low()
}

// ... 100 lines of other code ...

function low() { ... }

// Good - caller above callee, close together
function high() {
    medium()
}

function medium() {
    low()
}

function low() { ... }
```

### Variable Declarations

Declare variables as close to their usage as possible:

```pseudocode
// Bad - all declarations at top
function processOrder(order) {
    items = order.items
    customer = order.customer
    total = 0
    tax = 0
    discount = 0

    // ... 50 lines later, we use 'items' ...
    for item in items {
        total += item.price
    }

    // ... 20 lines later, we use 'tax' ...
    tax = total * TAX_RATE
}

// Good - declare near usage
function processOrder(order) {
    items = order.items
    total = 0
    for item in items {
        total += item.price
    }

    tax = total * TAX_RATE
    // ...
}
```

### Instance Variables

Declare at the top of the class (everyone knows where to look):

```pseudocode
class Order {
    // Instance variables at top
    id
    customer
    items
    status
    createdAt

    // Methods below
    function addItem(item) { ... }
    function calculateTotal() { ... }
}
```

### Dependent Functions

Caller should be above the callee:

```pseudocode
// Good - reads top-to-bottom
function processPayment(payment) {
    validatePayment(payment)
    chargeCard(payment)
    sendReceipt(payment)
}

function validatePayment(payment) {
    checkAmount(payment.amount)
    checkCard(payment.card)
}

function checkAmount(amount) { ... }
function checkCard(card) { ... }
function chargeCard(payment) { ... }
function sendReceipt(payment) { ... }
```

## Horizontal Formatting

### Line Length

Keep lines short. Common limits:
- Soft limit: 80-100 characters
- Hard limit: 120 characters
- Never: Horizontal scrolling

```pseudocode
// Bad - too long
function sendNotification(userId, messageType, messageContent, priority, scheduledTime, retryCount) { ... }

// Good - broken into readable lines
function sendNotification(
    userId,
    messageType,
    messageContent,
    priority,
    scheduledTime,
    retryCount
) {
    ...
}
```

### Horizontal Openness

Use spaces to accentuate relationships:

```pseudocode
// Operators - space around assignment, tight around high-precedence
total = price * quantity + tax
isValid = (a > b) && (c < d)

// Function calls - no space before parenthesis
calculateTotal(items)
user.getName()

// Separating arguments
function draw(x, y, width, height) { ... }
```

### Horizontal Alignment

Don't align variable declarations—it draws attention to the wrong thing:

```pseudocode
// Bad - over-aligned
private   name          = ""
private   age           = 0
private   emailAddress  = ""
private   phone         = ""

// Good - simple
private name = ""
private age = 0
private emailAddress = ""
private phone = ""
```

### Indentation

Respect the indentation hierarchy:

```pseudocode
// Good - clear structure
class User {
    name
    email

    function validate() {
        if (name.isEmpty()) {
            throw ValidationError("Name required")
        }

        if (!isValidEmail(email)) {
            throw ValidationError("Invalid email")
        }

        return true
    }
}

// Bad - collapsed
class User { name; email; function validate() { if (name.isEmpty()) { throw ValidationError("Name required") } if (!isValidEmail(email)) { throw ValidationError("Invalid email") } return true } }
```

### Don't Break Indentation for Short Statements

```pseudocode
// Bad
if (condition) return true

// Good
if (condition) {
    return true
}

// Bad
for (i in items) count++

// Good
for (i in items) {
    count++
}
```

## Team Rules

### Consistency Over Preference

A team should agree on formatting rules and follow them consistently:

- **Pick a style guide** and stick to it
- **Use automated formatters** (Prettier, Black, gofmt)
- **Enforce in CI** with linting
- **Don't debate** in code reviews—let tools handle it

### Common Team Decisions

| Decision | Options |
|----------|---------|
| Indentation | Tabs vs. spaces, 2 vs. 4 |
| Line length | 80, 100, or 120 |
| Braces | Same line vs. new line |
| Quotes | Single vs. double |
| Trailing commas | Yes vs. no |

### Automated Formatting

Use tools to eliminate debates:

```bash
# JavaScript/TypeScript
prettier --write .

# Python
black .

# Go
gofmt -w .

# PHP
php-cs-fixer fix .

# Rust
cargo fmt
```

## File Organization

### Standard Order

```pseudocode
// 1. File header/copyright
// Copyright 2024...

// 2. Imports/requires (grouped)
import standardLibrary
import thirdParty
import localModules

// 3. Constants
MAX_RETRIES = 3
DEFAULT_TIMEOUT = 30

// 4. Class definition
class MyClass {
    // 4a. Static/class variables
    static instances = 0

    // 4b. Instance variables
    private name
    private value

    // 4c. Constructor(s)
    constructor(name, value) { ... }

    // 4d. Public methods
    function doSomething() { ... }

    // 4e. Private methods
    private function helper() { ... }
}

// 5. Module exports (if applicable)
export MyClass
```

## Summary Checklist

### Vertical
- [ ] File is under 500 lines
- [ ] High-level concepts at top
- [ ] Blank lines separate concepts
- [ ] Related code is close together
- [ ] Variables declared near usage
- [ ] Caller above callee

### Horizontal
- [ ] Lines under 120 characters
- [ ] Consistent spacing around operators
- [ ] Proper indentation
- [ ] No horizontal alignment of declarations

### Team
- [ ] Agreed-upon style guide
- [ ] Automated formatting
- [ ] CI enforcement

---

*Based on concepts from "Clean Code" by Robert C. Martin.*
