# Error Handling

## Core Principle

> "Error handling is important, but if it obscures logic, it's wrong."
> — Robert C. Martin

Error handling should be clean and separate from main logic, not intertwined with it.

## Prefer Exceptions to Error Codes

Error codes force immediate handling and create nested structures.

```pseudocode
// Bad - error codes
function processFile(filename) {
    error = openFile(filename)
    if error == SUCCESS {
        error = readFile()
        if error == SUCCESS {
            error = parseContent()
            if error == SUCCESS {
                error = processContent()
                if error == SUCCESS {
                    return SUCCESS
                }
            }
        }
        closeFile()
    }
    return error
}

// Good - exceptions
function processFile(filename) {
    file = openFile(filename)
    try {
        content = readFile(file)
        data = parseContent(content)
        processContent(data)
    } finally {
        closeFile(file)
    }
}
```

## Write Try-Catch-Finally First

When writing code that might throw, start with the try-catch structure:

```pseudocode
// Start with the exception handling structure
function readFromFile(filename) {
    try {
        // Happy path goes here
        file = open(filename)
        return file.read()
    } catch (FileNotFound e) {
        throw new ConfigurationError("Config file missing: " + filename)
    } catch (PermissionDenied e) {
        throw new SecurityError("Cannot read file: " + filename)
    } finally {
        if file != null {
            file.close()
        }
    }
}
```

## Use Unchecked Exceptions

Checked exceptions (Java-style) violate the Open/Closed Principle—adding a new exception requires changing all callers.

```pseudocode
// Bad - checked exceptions cascade changes
function level1() throws ExceptionA, ExceptionB, ExceptionC {
    level2()
}

function level2() throws ExceptionA, ExceptionB, ExceptionC {
    level3()
}

function level3() throws ExceptionA, ExceptionB, ExceptionC {
    // Adding ExceptionD here requires changing level1, level2, level3
}

// Good - unchecked exceptions
function level1() {
    level2()
}

function level2() {
    level3()
}

function level3() {
    // Can throw new exceptions without changing callers
    throw new SomethingWentWrong()
}
```

## Provide Context with Exceptions

Include enough information to understand what failed:

```pseudocode
// Bad - no context
throw new Exception("Failed")

// Bad - generic message
throw new Exception("Error occurred")

// Good - specific context
throw new UserNotFoundException(
    "User not found with ID: " + userId +
    " in database: " + databaseName
)

// Good - custom exception with fields
class OrderProcessingException extends Exception {
    orderId
    stage
    reason

    constructor(orderId, stage, reason) {
        super("Order " + orderId + " failed at " + stage + ": " + reason)
        this.orderId = orderId
        this.stage = stage
        this.reason = reason
    }
}

throw new OrderProcessingException(
    "ORD-123",
    "payment",
    "Card declined"
)
```

## Define Exceptions by Caller's Needs

Classify exceptions by how the caller will handle them, not by source:

```pseudocode
// Bad - too many specific exceptions
try {
    port.open()
} catch (DeviceNotFound e) {
    reportError(e)
    log(e)
} catch (DeviceBusy e) {
    reportError(e)
    log(e)
} catch (DeviceTimeout e) {
    reportError(e)
    log(e)
} catch (InvalidConfig e) {
    reportError(e)
    log(e)
}

// Good - wrap in common exception type
class PortError extends Exception {
    constructor(message, cause) {
        super(message)
        this.cause = cause
    }
}

function openPort(port) {
    try {
        return port.open()
    } catch (DeviceNotFound | DeviceBusy | DeviceTimeout | InvalidConfig e) {
        throw new PortError("Cannot open port: " + port.name, e)
    }
}

// Caller has clean handling
try {
    openPort(port)
} catch (PortError e) {
    reportError(e)
    log(e)
}
```

## Use the Special Case Pattern

Instead of checking for null or special conditions, create objects that handle them:

```pseudocode
// Bad - null checks everywhere
function getEmployee(id) {
    employee = database.find(id)
    return employee  // Might be null
}

// Caller must check
employee = getEmployee(id)
if employee != null {
    pay = employee.calculatePay()
} else {
    pay = 0
}

// Good - special case object
class NullEmployee implements Employee {
    function calculatePay() {
        return 0
    }

    function getName() {
        return "Unknown"
    }

    function isNull() {
        return true
    }
}

function getEmployee(id) {
    employee = database.find(id)
    if employee == null {
        return new NullEmployee()
    }
    return employee
}

// Caller doesn't need null check
employee = getEmployee(id)
pay = employee.calculatePay()  // Always works
```

## Don't Return Null

Returning null forces callers to check for it, leading to errors:

```pseudocode
// Bad - returns null
function findUser(id) {
    user = database.find(id)
    return user  // Could be null
}

// Caller forgets to check -> NullPointerException
user = findUser(id)
sendEmail(user.email)  // Boom!

// Good - throw exception
function findUser(id) {
    user = database.find(id)
    if user == null {
        throw new UserNotFoundException(id)
    }
    return user
}

// Good - return Optional/Maybe
function findUser(id) {
    user = database.find(id)
    return Optional.ofNullable(user)
}

// Caller must handle explicitly
findUser(id).ifPresent(user => sendEmail(user.email))

// Good - return empty collection instead of null
function findUsersByRole(role) {
    users = database.findByRole(role)
    return users ?? []  // Never null, empty list OK
}
```

## Don't Pass Null

Passing null as an argument is even worse than returning it:

```pseudocode
// Bad - allows null parameters
function calculateDistance(point1, point2) {
    return sqrt((point2.x - point1.x)^2 + (point2.y - point1.y)^2)
}

// Caller passes null -> Crash
calculateDistance(null, point2)

// Good - validate parameters
function calculateDistance(point1, point2) {
    if point1 == null || point2 == null {
        throw new InvalidArgumentException("Points cannot be null")
    }
    return sqrt((point2.x - point1.x)^2 + (point2.y - point1.y)^2)
}

// Good - use assertions (in languages that support them)
function calculateDistance(point1, point2) {
    assert point1 != null
    assert point2 != null
    return sqrt((point2.x - point1.x)^2 + (point2.y - point1.y)^2)
}

// Best - use types that don't allow null
function calculateDistance(point1: Point, point2: Point) {
    // Type system guarantees non-null
    return sqrt((point2.x - point1.x)^2 + (point2.y - point1.y)^2)
}
```

## Separate Business Logic from Error Handling

```pseudocode
// Bad - mixed concerns
function processOrders(orders) {
    for order in orders {
        try {
            validateOrder(order)
            calculateTotals(order)
            chargeCustomer(order)
            fulfillOrder(order)
            notifyCustomer(order)
        } catch (ValidationError e) {
            log("Validation failed for order " + order.id + ": " + e.message)
            notifyAdmin(e)
        } catch (PaymentError e) {
            log("Payment failed for order " + order.id + ": " + e.message)
            refundIfNeeded(order)
            notifyCustomer(order, e)
        } catch (FulfillmentError e) {
            log("Fulfillment failed for order " + order.id + ": " + e.message)
            scheduleRetry(order)
        }
    }
}

// Good - separated
function processOrders(orders) {
    for order in orders {
        processOrderWithErrorHandling(order)
    }
}

function processOrderWithErrorHandling(order) {
    try {
        processOrder(order)
    } catch (OrderProcessingError e) {
        handleOrderError(order, e)
    }
}

function processOrder(order) {
    validateOrder(order)
    calculateTotals(order)
    chargeCustomer(order)
    fulfillOrder(order)
    notifyCustomer(order)
}

function handleOrderError(order, error) {
    log(error)
    switch error.type {
        case "validation": notifyAdmin(error)
        case "payment": handlePaymentFailure(order, error)
        case "fulfillment": scheduleRetry(order)
    }
}
```

## Summary

| Do | Don't |
|----|-------|
| Use exceptions | Use error codes |
| Provide context in exceptions | Throw generic exceptions |
| Wrap third-party exceptions | Leak implementation details |
| Use special case objects | Return null |
| Validate parameters | Allow null to propagate |
| Separate error handling from logic | Mix error handling with business logic |

---

*Based on concepts from "Clean Code" by Robert C. Martin.*
