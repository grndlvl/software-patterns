# Value Objects

## Definition

**Value Objects** are objects that describe aspects of the domain with no conceptual identity. They are defined entirely by their attributes rather than by a unique identifier. Value Objects represent descriptive characteristics of things in your domain.

Unlike Entities, Value Objects have no lifecycle to track and no continuity over time. Two Value Objects with the same attribute values are considered completely interchangeable.

## Key Characteristics

### 1. No Identity Needed

Value Objects don't need a unique identifier because they are defined by what they are, not by who they are:

```pseudocode
# Value Object - no ID needed
color1 = Color(red: 255, green: 0, blue: 0)
color2 = Color(red: 255, green: 0, blue: 0)

# These are equivalent - no need to track which is which
assert color1 == color2  # true
```

### 2. Immutability

Value Objects should be **immutable** - once created, their attributes cannot be changed. This makes them safe to share and eliminates many potential bugs:

```pseudocode
class Money:
    private readonly amount: Decimal
    private readonly currency: String

    constructor(amount: Decimal, currency: String):
        this.amount = amount
        this.currency = currency

    # No setters - attributes cannot be modified

    function getAmount(): Decimal
        return this.amount

    function getCurrency(): String
        return this.currency
```

### 3. Side-Effect Free Behavior

Operations on Value Objects should be **side-effect free functions** that return new Value Objects rather than modifying existing ones:

```pseudocode
class Money:
    # ...

    function add(other: Money): Money
        if this.currency != other.currency:
            throw new Error("Cannot add different currencies")

        # Return NEW Money object, don't modify this one
        return new Money(
            this.amount + other.amount,
            this.currency
        )

    function multiply(factor: Decimal): Money
        # Return NEW Money object
        return new Money(
            this.amount * factor,
            this.currency
        )
```

### 4. Equality by Attribute Comparison

Value Objects are equal if all their attributes are equal. Implement equality based on attribute values:

```pseudocode
class Address:
    private readonly street: String
    private readonly city: String
    private readonly postalCode: String
    private readonly country: String

    function equals(other: Address): Boolean
        return this.street == other.street
            AND this.city == other.city
            AND this.postalCode == other.postalCode
            AND this.country == other.country

    function hashCode(): Integer
        # Hash based on all attributes
        return hash(street, city, postalCode, country)
```

### 5. Replacing vs Modifying

Since Value Objects are immutable, you **replace** them rather than modify them:

```pseudocode
# DON'T do this (modification):
# customer.address.setStreet("123 New St")  # Not allowed!

# DO this (replacement):
newAddress = new Address(
    street: "123 New St",
    city: customer.address.getCity(),
    postalCode: customer.address.getPostalCode(),
    country: customer.address.getCountry()
)
customer.setAddress(newAddress)

# Or use a convenience method:
newAddress = customer.address.withStreet("123 New St")
customer.setAddress(newAddress)
```

## Benefits

### Simplicity
- No lifecycle to manage
- No identity tracking needed
- Easy to reason about

### Thread Safety
- Immutable objects are inherently thread-safe
- Can be shared freely across threads
- No synchronization needed

### Shareability
- Same Value Object instance can be safely shared
- No risk of unintended modifications
- Reduces memory footprint through sharing

### Conceptual Clarity
- Expresses domain concepts precisely
- Makes implicit concepts explicit
- Self-documenting code

## Common Value Objects

### Money

```pseudocode
class Money:
    private readonly amount: Decimal
    private readonly currency: String

    constructor(amount: Decimal, currency: String):
        if amount < 0:
            throw new Error("Amount cannot be negative")
        this.amount = amount
        this.currency = currency

    function add(other: Money): Money
        assertSameCurrency(other)
        return new Money(this.amount + other.amount, this.currency)

    function subtract(other: Money): Money
        assertSameCurrency(other)
        return new Money(this.amount - other.amount, this.currency)

    function multiply(factor: Decimal): Money
        return new Money(this.amount * factor, this.currency)

    function isGreaterThan(other: Money): Boolean
        assertSameCurrency(other)
        return this.amount > other.amount

    private function assertSameCurrency(other: Money):
        if this.currency != other.currency:
            throw new Error("Cannot operate on different currencies")

    function equals(other: Money): Boolean
        return this.amount == other.amount
            AND this.currency == other.currency
```

### Address

```pseudocode
class Address:
    private readonly street: String
    private readonly city: String
    private readonly state: String
    private readonly postalCode: String
    private readonly country: String

    constructor(street: String, city: String, state: String,
                postalCode: String, country: String):
        this.street = street
        this.city = city
        this.state = state
        this.postalCode = postalCode
        this.country = country

    # Convenience methods for creating modified copies
    function withStreet(newStreet: String): Address
        return new Address(newStreet, this.city, this.state,
                          this.postalCode, this.country)

    function withCity(newCity: String): Address
        return new Address(this.street, newCity, this.state,
                          this.postalCode, this.country)

    function toString(): String
        return "{street}, {city}, {state} {postalCode}, {country}"

    function equals(other: Address): Boolean
        return this.street == other.street
            AND this.city == other.city
            AND this.state == other.state
            AND this.postalCode == other.postalCode
            AND this.country == other.country
```

### DateRange

```pseudocode
class DateRange:
    private readonly start: Date
    private readonly end: Date

    constructor(start: Date, end: Date):
        if end < start:
            throw new Error("End date must be after start date")
        this.start = start
        this.end = end

    function contains(date: Date): Boolean
        return date >= this.start AND date <= this.end

    function overlaps(other: DateRange): Boolean
        return this.start <= other.end AND other.start <= this.end

    function durationInDays(): Integer
        return (this.end - this.start).days

    function expandBy(days: Integer): DateRange
        return new DateRange(
            this.start.minusDays(days),
            this.end.plusDays(days)
        )

    function equals(other: DateRange): Boolean
        return this.start == other.start AND this.end == other.end
```

### Email

```pseudocode
class Email:
    private readonly address: String

    constructor(address: String):
        if not isValidEmail(address):
            throw new Error("Invalid email format")
        this.address = address.toLowerCase()

    function getDomain(): String
        return this.address.split("@")[1]

    function getLocalPart(): String
        return this.address.split("@")[0]

    function toString(): String
        return this.address

    function equals(other: Email): Boolean
        return this.address == other.address

    private function isValidEmail(email: String): Boolean
        # Simple validation - use proper regex in real code
        return email.contains("@") AND email.contains(".")
```

## Design Patterns

### Making Implicit Concepts Explicit

Instead of primitive types scattered throughout code:

```pseudocode
# Before - primitive obsession
function processPayment(amount: Decimal, currency: String,
                       cardNumber: String, cardHolder: String): Boolean
    if amount <= 0:
        throw new Error("Invalid amount")
    if currency not in ["USD", "EUR", "GBP"]:
        throw new Error("Invalid currency")
    if cardNumber.length != 16:
        throw new Error("Invalid card number")
    # ...

# After - using Value Objects
function processPayment(amount: Money, card: CreditCard): Boolean
    # Validation already happened in Value Object constructors
    # Business logic is clearer
```

### Encapsulating Complex Validation

```pseudocode
class PhoneNumber:
    private readonly countryCode: String
    private readonly number: String

    constructor(countryCode: String, number: String):
        this.validateCountryCode(countryCode)
        this.validateNumber(number)
        this.countryCode = countryCode
        this.number = number

    private function validateCountryCode(code: String):
        if not code.matches("^\+[0-9]{1,3}$"):
            throw new Error("Invalid country code")

    private function validateNumber(num: String):
        if not num.matches("^[0-9]{7,15}$"):
            throw new Error("Invalid phone number")

    function toInternationalFormat(): String
        return "{countryCode} {number}"
```

### Bundling Related Attributes

```pseudocode
# Before - separate parameters
function createUser(firstName: String, lastName: String,
                   street: String, city: String, postalCode: String):
    # Many parameters, easy to mix up

# After - Value Objects
function createUser(name: PersonName, address: Address):
    # Clear, organized, hard to misuse

class PersonName:
    private readonly firstName: String
    private readonly lastName: String

    function getFullName(): String
        return "{firstName} {lastName}"
```

## When to Use Value Objects

### Use Value Objects When:
- The object is defined by its attributes, not by identity
- You need immutability for thread safety or sharing
- The concept appears repeatedly in your domain
- You want to encapsulate validation logic
- You need to bundle related attributes

### Don't Use Value Objects When:
- The object needs a unique identity
- The object has a lifecycle to track
- You need to modify attributes frequently (consider if design needs changing)
- The object is truly just a primitive value with no behavior

## Summary

| Aspect | Value Object | Entity |
|--------|--------------|--------|
| **Identity** | No unique identifier | Has unique identifier |
| **Equality** | By attribute values | By identity (ID) |
| **Mutability** | Immutable | Can be mutable |
| **Lifecycle** | No lifecycle tracking | Has lifecycle |
| **Sharing** | Safe to share instances | Usually not shared |
| **Example** | Money, Address, DateRange | Customer, Order, User |

## Key Takeaways

1. **Define by Attributes**: Value Objects are what they are based on their data, not who they are
2. **Make Immutable**: Once created, attributes cannot change
3. **Side-Effect Free**: Operations return new objects rather than modifying
4. **Equality by Value**: Two Value Objects with same attributes are equal
5. **Replace, Don't Modify**: Create new instances instead of changing existing ones
6. **Express Domain Concepts**: Make implicit concepts explicit through Value Objects
7. **Encapsulate Validation**: Put validation logic inside Value Object constructors
8. **Enable Sharing**: Immutability makes Value Objects safe to share

---

*Based on concepts from "Domain-Driven Design" by Eric Evans.*
