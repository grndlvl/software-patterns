# Decoupling and the Law of Demeter

## Overview

> "Good fences make good neighbors." - The Pragmatic Programmer

Decoupling is the practice of minimizing dependencies between modules or components in a system. Well-decoupled code is easier to change, test, and reason about because modifications to one part don't ripple through the entire system.

## The Law of Demeter (LoD)

The Law of Demeter, also known as the "Principle of Least Knowledge," states that an object should only talk to its immediate friends, not to strangers. Specifically, a method should only call:

1. **Methods on itself** (its own class)
2. **Methods on objects passed as parameters**
3. **Methods on objects it creates**
4. **Methods on instance variables** (its direct dependencies)

### What NOT to Do

Don't reach through objects to access distant collaborators:

```pseudocode
// VIOLATION: Reaching through multiple objects
function getCustomerCity(order):
    return order.getCustomer().getAddress().getCity()
```

This violates LoD because we're calling methods on objects we received from other calls (strangers).

### The Correct Approach

```pseudocode
// COMPLIANT: Ask the order directly
function getCustomerCity(order):
    return order.getCustomerCity()

// Inside Order class:
method getCustomerCity():
    return this.customer.getAddress().getCity()
```

Now the caller only talks to `order` (its immediate friend), and `order` handles the internal navigation.

## Train Wrecks

Train wrecks are long chains of method calls that couple your code to the entire chain of objects:

```pseudocode
// TRAIN WRECK: Fragile and tightly coupled
totalPrice = cart.getItems().get(0).getProduct().getPrice().getAmount()

// If any intermediate object changes, this breaks
```

### Fixing Train Wrecks

Ask for what you need, not how to get it:

```pseudocode
// BETTER: Tell, don't ask
totalPrice = cart.getFirstItemPrice()

// Or use a more intention-revealing method
totalPrice = cart.calculateFirstItemTotal()
```

## Shy Code

Shy code doesn't reveal its internals to the world. It maintains strong boundaries and communicates through well-defined interfaces.

### Example: Exposing Too Much

```pseudocode
// NOT SHY: Exposing internal structure
class Report:
    property data  // Public property
    property formatter  // Public property

    method generate():
        return this.formatter.format(this.data)

// Caller knows too much about internals
report = new Report()
report.formatter = new HTMLFormatter()
report.data = loadData()
output = report.generate()
```

### Example: Shy Code

```pseudocode
// SHY: Hidden internals, clear interface
class Report:
    private data
    private formatter

    method initialize(dataSource, format):
        this.data = dataSource
        this.formatter = FormatterFactory.create(format)

    method generate():
        return this.formatter.format(this.data)

// Caller only sees what it needs
report = new Report(dataSource, "html")
output = report.generate()
```

## How to Decouple

### 1. Use Interfaces/Protocols

```pseudocode
// Define contract, not implementation
interface PaymentProcessor:
    method processPayment(amount, account)

class CreditCardProcessor implements PaymentProcessor:
    method processPayment(amount, account):
        // Credit card specific logic

class PayPalProcessor implements PaymentProcessor:
    method processPayment(amount, account):
        // PayPal specific logic

// Client depends on interface, not concrete classes
class OrderService:
    private processor: PaymentProcessor

    method initialize(processor: PaymentProcessor):
        this.processor = processor

    method completeOrder(order):
        this.processor.processPayment(order.total, order.account)
```

### 2. Event-Driven Architecture

```pseudocode
// Publisher doesn't know about subscribers
class OrderSystem:
    private eventBus

    method placeOrder(order):
        // Process order
        order.status = "confirmed"

        // Publish event (no knowledge of listeners)
        this.eventBus.publish("order.placed", order)

// Subscribers register independently
class InventoryService:
    method initialize(eventBus):
        eventBus.subscribe("order.placed", this.handleOrderPlaced)

    method handleOrderPlaced(order):
        this.reduceStock(order.items)

class EmailService:
    method initialize(eventBus):
        eventBus.subscribe("order.placed", this.handleOrderPlaced)

    method handleOrderPlaced(order):
        this.sendConfirmationEmail(order.customer)
```

### 3. Message Passing

```pseudocode
// Components communicate via messages
class CustomerRepository:
    method findById(id):
        query = new FindCustomerQuery(id)
        return this.messageBus.send(query)

class CustomerQueryHandler:
    method handle(query: FindCustomerQuery):
        return this.database.fetchCustomer(query.customerId)

// No direct dependency between repository and handler
```

### 4. Dependency Injection

```pseudocode
// BAD: Hard-coded dependency
class ReportGenerator:
    method generate(data):
        database = new MySQLDatabase()  // Tight coupling!
        return database.fetch(data)

// GOOD: Injected dependency
class ReportGenerator:
    private database

    method initialize(database):
        this.database = database  // Any database implementation

    method generate(data):
        return this.database.fetch(data)
```

### 5. Wrapper/Adapter Pattern

```pseudocode
// Decouple from third-party libraries
interface Logger:
    method log(message, level)

class Log4jAdapter implements Logger:
    private log4j

    method initialize():
        this.log4j = new Log4jLogger()

    method log(message, level):
        this.log4j.write(message, level)

// Application depends on Logger interface, not Log4j
class Application:
    private logger: Logger

    method initialize(logger: Logger):
        this.logger = logger
```

## Benefits of Decoupling

| Benefit | Description |
|---------|-------------|
| **Testability** | Mock or stub dependencies easily; test components in isolation |
| **Flexibility** | Swap implementations without changing client code |
| **Maintainability** | Changes localized to single components; reduced ripple effects |
| **Reusability** | Components work independently; can be used in different contexts |
| **Parallel Development** | Teams work on different components without blocking each other |
| **Resilience** | Failures contained; don't cascade through the system |
| **Understandability** | Each component has clear, minimal responsibilities |

## Recognizing Coupling

### Signs of Tight Coupling

```pseudocode
// Multiple dots in a call chain
result = object.getA().getB().getC().doSomething()

// Methods that only delegate to other objects
function processOrder(order):
    order.getCustomer().getAccount().debit(order.getTotal())
    order.getInventory().reduce(order.getItems())

// Classes that know too much about other classes' structure
function validateUser(user):
    if user.profile.settings.notifications.email.enabled:
        // Too much knowledge about user's internal structure
```

### Measuring Coupling

- **Afferent Coupling (Ca)**: Number of classes that depend on this class
- **Efferent Coupling (Ce)**: Number of classes this class depends on
- **Instability (I)**: Ce / (Ca + Ce) — ranges from 0 (stable) to 1 (unstable)

Aim for balanced coupling: stable interfaces with unstable implementations.

## Summary Table

| Principle | What It Means | Example |
|-----------|---------------|---------|
| **Law of Demeter** | Only talk to immediate friends | `order.getPrice()` not `order.getCustomer().getAddress().getCity()` |
| **Avoid Train Wrecks** | No long method chains | Use delegation or queries instead |
| **Shy Code** | Hide internals, expose behavior | Private data, public methods |
| **Interface-Based Design** | Depend on contracts, not implementations | `PaymentProcessor` interface, not `CreditCardProcessor` |
| **Event-Driven** | Publish events, don't call directly | Event bus instead of direct method calls |
| **Dependency Injection** | Pass dependencies in, don't create them | Constructor/method injection |
| **Message Passing** | Communicate via messages | Commands, queries, events |
| **Wrappers** | Isolate third-party code | Adapter pattern for external libraries |

## Practical Guidelines

1. **Ask, Don't Reach**: If you need data from a distant object, add a method to ask for it
2. **Tell, Don't Ask**: Instead of getting data and acting on it, tell the object what to do
3. **One Dot Rule**: Generally limit chained calls (exceptions for fluent APIs)
4. **Use Events**: For one-to-many relationships, use events rather than direct calls
5. **Inject Dependencies**: Let configuration/DI container wire collaborators
6. **Program to Interfaces**: Depend on abstractions, not concrete classes
7. **Encapsulate Collections**: Don't expose internal collections; provide methods to work with them
8. **Minimize Knowledge**: Each component should know as little as possible about others

---

*Based on concepts from "The Pragmatic Programmer" by David Thomas and Andrew Hunt.*
