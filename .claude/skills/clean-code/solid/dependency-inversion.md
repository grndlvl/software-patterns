# Dependency Inversion Principle (DIP)

## Definition

> "High-level modules should not depend on low-level modules. Both should depend on abstractions."
> "Abstractions should not depend on details. Details should depend on abstractions."
> — Robert C. Martin

Depend on interfaces, not concrete implementations. The direction of dependency should be toward stability and abstraction.

## The Problem

When high-level modules depend on low-level modules:
- Changes to low-level modules break high-level modules
- High-level business logic is coupled to infrastructure details
- Code becomes hard to test (can't mock dependencies)
- Switching implementations requires changing core code

## Recognizing Violations

**Warning signs:**
- Business logic imports database/file/network classes directly
- `new ConcreteClass()` scattered throughout code
- Hard to test without real database/services
- Changing a library requires modifying business code

## Examples

### Bad: Direct Dependencies

```pseudocode
class OrderService {
    function createOrder(orderData) {
        // Directly depends on concrete MySQL class
        database = new MySQLDatabase()
        database.connect("localhost", "root", "password")

        // Directly depends on concrete email class
        emailer = new SendGridEmailer()
        emailer.setApiKey("xxx")

        // Business logic...
        order = new Order(orderData)
        database.insert("orders", order)
        emailer.send(order.customerEmail, "Order confirmed!")

        return order
    }
}

// Problems:
// - Can't test without real MySQL and SendGrid
// - Changing to PostgreSQL requires modifying OrderService
// - Business logic mixed with infrastructure details
```

### Good: Dependency Inversion

```pseudocode
interface Database {
    function insert(table, data)
    function find(table, criteria)
}

interface Emailer {
    function send(to, message)
}

class OrderService {
    database: Database
    emailer: Emailer

    constructor(database: Database, emailer: Emailer) {
        this.database = database
        this.emailer = emailer
    }

    function createOrder(orderData) {
        // Business logic only - no infrastructure details
        order = new Order(orderData)
        database.insert("orders", order)
        emailer.send(order.customerEmail, "Order confirmed!")
        return order
    }
}

// Concrete implementations
class MySQLDatabase implements Database {
    function insert(table, data) { /* MySQL specific */ }
    function find(table, criteria) { /* MySQL specific */ }
}

class PostgresDatabase implements Database {
    function insert(table, data) { /* Postgres specific */ }
    function find(table, criteria) { /* Postgres specific */ }
}

class SendGridEmailer implements Emailer {
    function send(to, message) { /* SendGrid API */ }
}

class SMTPEmailer implements Emailer {
    function send(to, message) { /* SMTP protocol */ }
}

// Composition at application startup
database = new MySQLDatabase()
emailer = new SendGridEmailer()
orderService = new OrderService(database, emailer)

// Easy to test with fakes
class FakeDatabase implements Database {
    insertedData = []
    function insert(table, data) { insertedData.append({table, data}) }
    function find(table, criteria) { return [] }
}

class FakeEmailer implements Emailer {
    sentEmails = []
    function send(to, message) { sentEmails.append({to, message}) }
}

// Test
function testCreateOrder() {
    fakeDb = new FakeDatabase()
    fakeEmail = new FakeEmailer()
    service = new OrderService(fakeDb, fakeEmail)

    service.createOrder({product: "Book", customer: "test@example.com"})

    assert(fakeDb.insertedData.length == 1)
    assert(fakeEmail.sentEmails.length == 1)
}
```

### Bad: Hardcoded Logger

```pseudocode
class PaymentProcessor {
    function processPayment(payment) {
        logger = new FileLogger("/var/log/payments.log")

        logger.info("Processing payment: " + payment.id)

        // Process payment...

        logger.info("Payment completed: " + payment.id)
    }
}
```

### Good: Injected Logger

```pseudocode
interface Logger {
    function info(message)
    function error(message)
}

class PaymentProcessor {
    logger: Logger

    constructor(logger: Logger) {
        this.logger = logger
    }

    function processPayment(payment) {
        logger.info("Processing payment: " + payment.id)
        // Process payment...
        logger.info("Payment completed: " + payment.id)
    }
}

// Different loggers for different environments
class FileLogger implements Logger { /* ... */ }
class CloudWatchLogger implements Logger { /* ... */ }
class ConsoleLogger implements Logger { /* ... */ }
class NullLogger implements Logger {
    function info(message) { }  // Silent for tests
    function error(message) { }
}
```

## Dependency Injection Patterns

### Constructor Injection (Preferred)

```pseudocode
class UserService {
    repository: UserRepository
    hasher: PasswordHasher

    constructor(repository: UserRepository, hasher: PasswordHasher) {
        this.repository = repository
        this.hasher = hasher
    }
}

// Dependencies explicit and required
service = new UserService(new MySQLUserRepository(), new BCryptHasher())
```

### Setter Injection

```pseudocode
class ReportGenerator {
    formatter: ReportFormatter

    function setFormatter(formatter: ReportFormatter) {
        this.formatter = formatter
    }

    function generate(data) {
        if formatter == null {
            throw "Formatter not set"
        }
        return formatter.format(data)
    }
}

// Allows optional or late-bound dependencies
generator = new ReportGenerator()
generator.setFormatter(new PDFFormatter())
```

### Interface Injection

```pseudocode
interface FormatterAware {
    function setFormatter(formatter: Formatter)
}

class Report implements FormatterAware {
    formatter

    function setFormatter(formatter: Formatter) {
        this.formatter = formatter
    }
}
```

## Inversion of Control (IoC) Containers

Automate dependency wiring:

```pseudocode
// Register dependencies
container.register(Database, MySQLDatabase)
container.register(Emailer, SendGridEmailer)
container.register(Logger, FileLogger)
container.register(OrderService)  // Auto-injects dependencies

// Resolve with all dependencies injected
orderService = container.resolve(OrderService)
// OrderService gets MySQLDatabase, SendGridEmailer automatically
```

## The Dependency Rule

Dependencies should point inward toward business logic:

```
┌─────────────────────────────────────────────┐
│                 Frameworks                   │
│  ┌─────────────────────────────────────┐    │
│  │            Interfaces/Adapters       │    │
│  │  ┌─────────────────────────────┐    │    │
│  │  │       Use Cases/Services     │    │    │
│  │  │  ┌─────────────────────┐    │    │    │
│  │  │  │      Entities        │    │    │    │
│  │  │  │   (Core Business)    │    │    │    │
│  │  │  └─────────────────────┘    │    │    │
│  │  └─────────────────────────────┘    │    │
│  └─────────────────────────────────────┘    │
└─────────────────────────────────────────────┘

Inner layers know NOTHING about outer layers
Dependencies always point INWARD
```

## Stable vs Volatile Dependencies

**Stable** (OK to depend on directly):
- Language primitives (String, Int)
- Standard library collections
- Well-established libraries

**Volatile** (inject via interface):
- Database connections
- File system access
- Network calls
- External services
- Time/random (for testability)

```pseudocode
// Clock interface for testable time
interface Clock {
    function now(): DateTime
}

class SystemClock implements Clock {
    function now() { return DateTime.now() }
}

class FakeClock implements Clock {
    currentTime

    function now() { return currentTime }
    function advance(duration) { currentTime += duration }
}

// Now time-dependent code is testable
class SubscriptionService {
    clock: Clock

    function isExpired(subscription) {
        return subscription.expiresAt < clock.now()
    }
}
```

## Benefits

- **Testability**: Inject fakes/mocks for testing
- **Flexibility**: Swap implementations without changing code
- **Maintainability**: Changes isolated to specific implementations
- **Parallel development**: Teams work on interfaces independently
- **Cleaner architecture**: Business logic free of infrastructure

## Related Principles

- **Open/Closed**: DIP enables extension without modification
- **Interface Segregation**: Depend on focused interfaces
- **Single Responsibility**: Separate concerns via injection

---

*Based on concepts from "Clean Code" and "Agile Software Development" by Robert C. Martin.*
