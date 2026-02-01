# Functions

## Core Principle

> "Functions should do one thing. They should do it well. They should do it only."
> — Robert C. Martin

Functions are the first line of organization in any program. Writing them well makes code readable and maintainable.

## Rules for Clean Functions

### 1. Keep Functions Small

Functions should be small. Then they should be smaller than that.

```pseudocode
// Bad - too long, does multiple things
function processOrder(order) {
    // Validate order (10 lines)
    if order.items.isEmpty() { throw EmptyOrderError }
    if order.customer == null { throw NoCustomerError }
    for item in order.items {
        if item.quantity <= 0 { throw InvalidQuantityError }
        if not inventory.hasStock(item.productId, item.quantity) {
            throw OutOfStockError
        }
    }

    // Calculate totals (15 lines)
    subtotal = 0
    for item in order.items {
        subtotal += item.price * item.quantity
    }
    tax = subtotal * taxRate
    shipping = calculateShipping(order)
    total = subtotal + tax + shipping

    // Save order (10 lines)
    order.subtotal = subtotal
    order.tax = tax
    order.shipping = shipping
    order.total = total
    order.status = "PENDING"
    database.save(order)

    // Send notifications (10 lines)
    emailService.send(order.customer.email, "Order Confirmed", ...)
    if order.total > 1000 {
        slackService.notify("#high-value-orders", ...)
    }

    return order
}

// Good - small, focused functions
function processOrder(order) {
    validateOrder(order)
    calculateOrderTotals(order)
    saveOrder(order)
    sendOrderNotifications(order)
    return order
}

function validateOrder(order) {
    ensureOrderHasItems(order)
    ensureOrderHasCustomer(order)
    validateItemQuantities(order)
    validateInventory(order)
}

function calculateOrderTotals(order) {
    order.subtotal = calculateSubtotal(order.items)
    order.tax = calculateTax(order.subtotal)
    order.shipping = calculateShipping(order)
    order.total = order.subtotal + order.tax + order.shipping
}
```

### 2. Do One Thing

A function should do one thing, do it well, and do it only.

**Test**: Can you extract another function with a name that's not a restatement?

```pseudocode
// Bad - does multiple things
function emailClients(clients) {
    for client in clients {
        clientRecord = database.find(client.id)
        if clientRecord.isActive() {
            email(client)
        }
    }
}

// Good - each function does one thing
function emailActiveClients(clients) {
    activeClients = filterActiveClients(clients)
    for client in activeClients {
        emailClient(client)
    }
}

function filterActiveClients(clients) {
    return clients.filter(client => isActiveClient(client))
}

function isActiveClient(client) {
    record = database.find(client.id)
    return record.isActive()
}
```

### 3. One Level of Abstraction

Statements within a function should be at the same level of abstraction.

```pseudocode
// Bad - mixed abstraction levels
function analyze(text) {
    // High level
    document = parseDocument(text)

    // Low level
    words = text.split(" ")
    wordCount = words.length

    // High level
    sentiment = analyzeSentiment(document)

    // Low level
    uniqueWords = new Set()
    for word in words {
        uniqueWords.add(word.toLowerCase())
    }

    return new Analysis(wordCount, uniqueWords.size(), sentiment)
}

// Good - consistent abstraction level
function analyze(text) {
    document = parseDocument(text)
    wordCount = countWords(document)
    uniqueWordCount = countUniqueWords(document)
    sentiment = analyzeSentiment(document)
    return new Analysis(wordCount, uniqueWordCount, sentiment)
}
```

### 4. Use Descriptive Names

Long descriptive names are better than short cryptic ones.

```pseudocode
// Bad
function calc(a, b, t) { ... }

// Good
function calculateMonthlyPayment(principal, annualRate, termInYears) { ... }
```

```pseudocode
// Bad
function process() { ... }

// Good
function processExpiredSubscriptions() { ... }
function sendRenewalReminders() { ... }
```

### 5. Minimize Arguments

The ideal number of arguments is zero. Then one, then two. Three should be avoided.

```pseudocode
// Bad - too many arguments
function createUser(name, email, password, age, address, phone, role) { ... }

// Good - use an object
function createUser(userData) {
    // userData contains all user properties
}

// Or use a builder
user = UserBuilder.new()
    .withName("John")
    .withEmail("john@example.com")
    .withRole("admin")
    .build()
```

#### Argument Patterns

**Niladic (0 args)**: Best when possible
```pseudocode
function now() { return DateTime.current() }
```

**Monadic (1 arg)**: Three common forms
```pseudocode
// Asking a question
function isValidEmail(email) { ... }

// Transforming
function parseJson(jsonString) { ... }

// Event (no return)
function logError(message) { ... }
```

**Dyadic (2 args)**: Acceptable when ordered naturally
```pseudocode
function add(a, b) { ... }
function drawPoint(x, y) { ... }
```

**Triadic (3 args)**: Avoid when possible
```pseudocode
// Bad
function createRange(start, end, step) { ... }

// Better
function createRange(rangeSpec) { ... }
// Or
range = Range.from(start).to(end).step(step)
```

### 6. Avoid Flag Arguments

Boolean arguments indicate the function does more than one thing.

```pseudocode
// Bad - flag changes behavior
function createFile(name, temporary) {
    if temporary {
        // Create temp file
    } else {
        // Create permanent file
    }
}

// Good - separate functions
function createFile(name) { ... }
function createTemporaryFile(name) { ... }
```

### 7. No Side Effects

Functions should do what they say—nothing more.

```pseudocode
// Bad - hidden side effect
function validatePassword(username, password) {
    user = database.find(username)
    if user.password == hash(password) {
        Session.initialize()  // Hidden side effect!
        return true
    }
    return false
}

// Good - no hidden effects
function validatePassword(username, password) {
    user = database.find(username)
    return user.password == hash(password)
}

// Separate function for session
function loginUser(username, password) {
    if validatePassword(username, password) {
        Session.initialize()
        return true
    }
    return false
}
```

### 8. Command Query Separation

Functions should either do something (command) or answer something (query), not both.

```pseudocode
// Bad - does both
function setAndCheckAttribute(name, value) {
    if attributeExists(name) {
        setAttribute(name, value)
        return true
    }
    return false
}

// Usage is confusing
if setAndCheckAttribute("username", "bob") { ... }

// Good - separate command and query
function attributeExists(name) {
    return attributes.has(name)
}

function setAttribute(name, value) {
    attributes.set(name, value)
}

// Clear usage
if attributeExists("username") {
    setAttribute("username", "bob")
}
```

### 9. Prefer Exceptions to Error Codes

Error codes force immediate handling and lead to nested structures.

```pseudocode
// Bad - error codes
function deletePage(page) {
    if deletePageAndReferences(page) == OK {
        if registry.deleteReference(page.name) == OK {
            if configKeys.deleteKey(page.name.makeKey()) == OK {
                log("Page deleted")
            } else {
                log("Config key not deleted")
            }
        } else {
            log("Reference not deleted")
        }
    } else {
        log("Delete failed")
    }
}

// Good - exceptions
function deletePage(page) {
    try {
        deletePageAndReferences(page)
        registry.deleteReference(page.name)
        configKeys.deleteKey(page.name.makeKey())
        log("Page deleted")
    } catch (Exception e) {
        log("Delete failed: " + e.message)
    }
}
```

### 10. Extract Try/Catch Blocks

Error handling is one thing—extract it.

```pseudocode
// Bad - mixed concerns
function delete(page) {
    try {
        page.validateDeletion()
        page.deleteContent()
        page.removeFromIndex()
        page.notifySubscribers()
    } catch (Exception e) {
        logError(e)
    }
}

// Good - separated
function delete(page) {
    try {
        deletePageInternal(page)
    } catch (Exception e) {
        logError(e)
    }
}

function deletePageInternal(page) {
    page.validateDeletion()
    page.deleteContent()
    page.removeFromIndex()
    page.notifySubscribers()
}
```

## The Stepdown Rule

Code should read like a top-down narrative. Each function should be followed by functions at the next level of abstraction.

```pseudocode
// Read top-to-bottom like a story
function processPayroll() {
    employees = getActiveEmployees()
    for employee in employees {
        pay = calculatePay(employee)
        recordPayment(employee, pay)
        sendPayStub(employee, pay)
    }
}

function getActiveEmployees() {
    return employeeRepository.findActive()
}

function calculatePay(employee) {
    basePay = calculateBasePay(employee)
    overtime = calculateOvertime(employee)
    deductions = calculateDeductions(employee)
    return basePay + overtime - deductions
}

function calculateBasePay(employee) {
    return employee.salary / PAY_PERIODS_PER_YEAR
}

// ... and so on, drilling down into details
```

## Summary Checklist

- [ ] Is the function small (≤20 lines)?
- [ ] Does it do exactly one thing?
- [ ] Are all statements at the same abstraction level?
- [ ] Does the name describe what it does?
- [ ] Does it have few arguments (≤2)?
- [ ] Are there no flag arguments?
- [ ] Are there no side effects?
- [ ] Does it follow command-query separation?
- [ ] Does error handling have its own function?

---

*Based on concepts from "Clean Code" by Robert C. Martin.*
