# Open/Closed Principle (OCP)

## Definition

> "Software entities (classes, modules, functions) should be open for extension, but closed for modification."
> — Bertrand Meyer (popularized by Robert C. Martin)

You should be able to add new functionality without changing existing code.

## The Problem

When you modify existing code to add features:
- Risk of breaking working functionality
- Existing tests may fail
- Other code depending on it may break
- Changes cascade through the system

## Recognizing Violations

**Warning signs:**
- Adding a new type requires modifying existing switch/if statements
- New features require changes to core classes
- You're frequently editing the same class for different features
- Type checking with `instanceof` or `typeof`

## Examples

### Bad: Modification Required

```pseudocode
class AreaCalculator {
    function calculateArea(shape) {
        if shape.type == "circle" {
            return PI * shape.radius * shape.radius
        } else if shape.type == "rectangle" {
            return shape.width * shape.height
        } else if shape.type == "triangle" {
            return 0.5 * shape.base * shape.height
        }
        // Adding a new shape requires modifying this class!
    }
}

// To add a pentagon, we must MODIFY AreaCalculator
```

### Good: Extension Without Modification

```pseudocode
interface Shape {
    function area()
}

class Circle implements Shape {
    radius

    function area() {
        return PI * radius * radius
    }
}

class Rectangle implements Shape {
    width
    height

    function area() {
        return width * height
    }
}

class Triangle implements Shape {
    base
    height

    function area() {
        return 0.5 * base * height
    }
}

class AreaCalculator {
    function calculateArea(shape: Shape) {
        return shape.area()  // Works for ANY shape
    }
}

// To add a pentagon, just create a new class - no modifications needed
class Pentagon implements Shape {
    side

    function area() {
        return 0.25 * sqrt(5 * (5 + 2 * sqrt(5))) * side * side
    }
}
```

### Bad: Switch on Type

```pseudocode
class PaymentProcessor {
    function process(payment) {
        switch payment.type {
            case "credit_card":
                return processCreditCard(payment)
            case "paypal":
                return processPayPal(payment)
            case "bitcoin":
                return processBitcoin(payment)
            // Every new payment method = modify this class
        }
    }
}
```

### Good: Strategy Pattern

```pseudocode
interface PaymentMethod {
    function process(amount)
}

class CreditCardPayment implements PaymentMethod {
    function process(amount) {
        // Credit card specific logic
        return gateway.charge(cardNumber, amount)
    }
}

class PayPalPayment implements PaymentMethod {
    function process(amount) {
        // PayPal specific logic
        return paypal.sendPayment(email, amount)
    }
}

class PaymentProcessor {
    function process(payment: PaymentMethod, amount) {
        return payment.process(amount)
    }
}

// Adding Stripe? Just create a new class
class StripePayment implements PaymentMethod {
    function process(amount) {
        return stripe.charge(token, amount)
    }
}
```

### Bad: Hardcoded Discounts

```pseudocode
class PriceCalculator {
    function calculatePrice(product, customerType) {
        price = product.basePrice

        if customerType == "regular" {
            return price
        } else if customerType == "premium" {
            return price * 0.9  // 10% off
        } else if customerType == "vip" {
            return price * 0.8  // 20% off
        }
    }
}
```

### Good: Extensible Discounts

```pseudocode
interface DiscountStrategy {
    function apply(price)
}

class NoDiscount implements DiscountStrategy {
    function apply(price) {
        return price
    }
}

class PercentageDiscount implements DiscountStrategy {
    percentage

    function apply(price) {
        return price * (1 - percentage / 100)
    }
}

class FixedDiscount implements DiscountStrategy {
    amount

    function apply(price) {
        return max(0, price - amount)
    }
}

class PriceCalculator {
    function calculatePrice(product, discount: DiscountStrategy) {
        return discount.apply(product.basePrice)
    }
}

// New discount type? No modification needed
class BuyOneGetOneFree implements DiscountStrategy {
    function apply(price) {
        return price / 2
    }
}
```

## Techniques for OCP

### 1. Polymorphism
Use interfaces and inheritance to allow new implementations.

### 2. Strategy Pattern
Encapsulate algorithms in separate classes.

### 3. Template Method Pattern
Define skeleton in base class, let subclasses fill in steps.

### 4. Decorator Pattern
Add behavior by wrapping objects.

### 5. Plugin Architecture
Load new functionality at runtime.

```pseudocode
// Plugin system example
class ReportGenerator {
    formatters = []

    function registerFormatter(formatter: ReportFormatter) {
        formatters.add(formatter)
    }

    function generate(data, format) {
        formatter = formatters.find(f => f.supports(format))
        return formatter.format(data)
    }
}

// Add new formats without modifying ReportGenerator
generator.registerFormatter(new PDFFormatter())
generator.registerFormatter(new ExcelFormatter())
generator.registerFormatter(new HTMLFormatter())
```

## When Modification is OK

OCP doesn't mean never modify code. Modification is acceptable for:
- **Bug fixes**: Fixing broken behavior
- **Refactoring**: Improving structure without changing behavior
- **Breaking changes**: When extension isn't feasible

The goal is to *design* for extension so that *new features* don't require modification.

## Balancing Act

Don't over-engineer. Apply OCP when:
- You expect variation (multiple payment types, report formats)
- Changes are requested frequently in an area
- The cost of modification is high

Don't apply OCP when:
- Requirements are stable
- Abstraction adds more complexity than value
- YAGNI (You Aren't Gonna Need It)

## Benefits

- **Stability**: Existing code remains untouched and tested
- **Safety**: New features can't break old ones
- **Parallel work**: Teams can add features independently
- **Testing**: Only new code needs new tests

## Related Patterns

- **Strategy**: Encapsulate interchangeable algorithms
- **Decorator**: Add behavior without modification
- **Factory**: Create objects without specifying concrete types
- **Template Method**: Define skeleton, extend steps

---

*Based on concepts from "Clean Code" and "Agile Software Development" by Robert C. Martin.*
