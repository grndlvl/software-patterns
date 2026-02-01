# Orthogonality

## Definition

> "Two or more things are orthogonal if changes in one do not affect any of the others."
> — Dave Thomas & Andy Hunt

In computing, orthogonality means independence—components that don't affect each other when changed.

## The Geometry Metaphor

In geometry, orthogonal means at right angles. Move along one axis and your position on other axes doesn't change.

```
Y
│
│    Moving along X doesn't change Y
│    Moving along Y doesn't change X
└────────── X
```

## Benefits of Orthogonality

### Increased Productivity

- Changes are localized
- Components can be combined in new ways
- Testing is easier
- Less duplication

### Reduced Risk

- Diseased sections are isolated
- System is less fragile
- Components can be replaced
- Easier to test parts in isolation

## Orthogonal Design

### Layered Architecture

```
┌─────────────────────────┐
│    Presentation Layer   │  ← Changes don't affect data layer
├─────────────────────────┤
│    Business Logic       │  ← Independent of UI and database
├─────────────────────────┤
│    Data Access Layer    │  ← Changes don't affect business logic
├─────────────────────────┤
│    Database             │
└─────────────────────────┘
```

### Non-Orthogonal Example

```pseudocode
// Bad - UI logic mixed with business logic
class OrderButton {
    function onClick() {
        // UI code
        button.disable()
        spinner.show()

        // Business logic (non-orthogonal!)
        order.validate()
        order.calculateTotal()
        order.applyDiscounts()

        // Database code (non-orthogonal!)
        database.save(order)

        // More UI code
        spinner.hide()
        showSuccessMessage()
    }
}
```

### Orthogonal Example

```pseudocode
// Good - separated concerns
class OrderButton {
    function onClick() {
        ui.showLoading()
        orderService.processOrder(order)
            .then(() => ui.showSuccess())
            .catch(e => ui.showError(e))
    }
}

class OrderService {
    function processOrder(order) {
        validator.validate(order)
        calculator.calculateTotal(order)
        repository.save(order)
    }
}

class OrderRepository {
    function save(order) {
        database.insert("orders", order)
    }
}

// Now: UI changes don't affect business logic
// Business logic changes don't affect database
```

## Testing Orthogonality

Ask these questions:

1. **If I dramatically change X, how many modules are affected?**
   - Good: Only one
   - Bad: Many

2. **Can I test this module in isolation?**
   - Good: Yes, with simple mocks
   - Bad: Need complex setup

3. **Do my modules know about each other's internals?**
   - Good: No, only interfaces
   - Bad: Yes, tightly coupled

## Achieving Orthogonality

### Keep Code Decoupled

```pseudocode
// Bad - objects know too much about each other
class Order {
    function ship() {
        customer.address.city.warehouse.stock.reduce(items)
        customer.loyaltyPoints.add(total * 0.1)
        emailService.templates.orderShipped.send(customer.email)
    }
}

// Good - tell, don't ask
class Order {
    function ship(shippingService) {
        shippingService.ship(this)
    }
}

class ShippingService {
    function ship(order) {
        warehouse.reduceStock(order.items)
        loyalty.addPoints(order.customer, order.total)
        notifications.sendShipped(order)
    }
}
```

### Avoid Global Data

```pseudocode
// Bad - global state couples everything
global currentUser
global settings
global database

function processOrder() {
    if settings.taxEnabled {
        tax = currentUser.region.taxRate * total
    }
    database.save(order)  // Which database? Can't test!
}

// Good - inject dependencies
function processOrder(order, user, settings, database) {
    if settings.taxEnabled {
        tax = calculateTax(user.region, total)
    }
    database.save(order)
}
```

### Avoid Similar Functions

If two functions look similar, they might share code that should be extracted—but be careful not to couple unrelated things.

```pseudocode
// These are similar but serve different domains
function formatUserName(user) {
    return user.firstName + " " + user.lastName
}

function formatProductName(product) {
    return product.brand + " " + product.model
}

// DON'T create: formatName(thing, prop1, prop2)
// They might evolve differently and should remain orthogonal
```

## Orthogonality in Teams

### Bad: Component-Based Teams

```
Team A: Database layer (all tables)
Team B: UI layer (all screens)
Team C: Business logic (all features)

Problem: One feature requires all three teams to coordinate
```

### Good: Feature-Based Teams

```
Team A: User management (UI + logic + data)
Team B: Ordering (UI + logic + data)
Team C: Inventory (UI + logic + data)

Benefit: Each team can work independently
```

## Orthogonality Checklist

When designing components, ask:

- [ ] Can I change this without affecting others?
- [ ] Can I test this in isolation?
- [ ] Does this have a single, well-defined purpose?
- [ ] Are dependencies injected, not hardcoded?
- [ ] Would a change ripple through the system?

## Real-World Examples

### Orthogonal

- **Unix pipes**: `cat file | grep pattern | sort | uniq`
- **CSS classes**: Multiple classes combine independently
- **Microservices**: Services deployed independently
- **Plugins**: Add features without changing core

### Non-Orthogonal

- **Spaghetti code**: Everything depends on everything
- **God objects**: One class that does everything
- **Tight coupling**: Can't change A without changing B
- **Feature flags everywhere**: Logic scattered across codebase

## The Helicopter Crash

The book tells a story of a helicopter crash investigation:

- Cause: A single leak in hydraulic system
- Root cause: Both control systems shared the same hydraulic lines
- Result: One failure brought down everything

**Lesson**: Keep systems truly independent. Don't let them share failure modes.

## Summary

| Orthogonal | Non-Orthogonal |
|------------|----------------|
| Layered architecture | Spaghetti code |
| Dependency injection | Global variables |
| Interface segregation | Fat interfaces |
| Single responsibility | God objects |
| Feature teams | Component teams |
| Microservices | Monolith with no boundaries |

---

*Based on concepts from "The Pragmatic Programmer" by David Thomas and Andrew Hunt.*
