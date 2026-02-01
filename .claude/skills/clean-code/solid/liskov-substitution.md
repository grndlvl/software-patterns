# Liskov Substitution Principle (LSP)

## Definition

> "Objects of a superclass should be replaceable with objects of its subclasses without breaking the application."
> — Barbara Liskov

If S is a subtype of T, then objects of type T can be replaced with objects of type S without altering the correctness of the program.

## The Problem

When subclasses don't properly substitute for their parents:
- Code that works with the base class breaks with derived classes
- Polymorphism becomes unreliable
- Clients need type checking to handle special cases
- The inheritance hierarchy is misleading

## Recognizing Violations

**Warning signs:**
- Subclass throws exceptions the parent doesn't
- Subclass has empty or no-op implementations of parent methods
- Subclass requires type checking before use
- Subclass violates parent's documented behavior
- Square extends Rectangle problems

## The Classic Example: Rectangle and Square

### Bad: Square Violates LSP

```pseudocode
class Rectangle {
    width
    height

    function setWidth(w) {
        width = w
    }

    function setHeight(h) {
        height = h
    }

    function area() {
        return width * height
    }
}

class Square extends Rectangle {
    function setWidth(w) {
        width = w
        height = w  // Must keep square!
    }

    function setHeight(h) {
        width = h   // Must keep square!
        height = h
    }
}

// This code breaks with Square
function testRectangle(rect: Rectangle) {
    rect.setWidth(5)
    rect.setHeight(4)
    assert(rect.area() == 20)  // Fails for Square! Area = 16
}
```

### Good: Separate Abstractions

```pseudocode
interface Shape {
    function area()
}

class Rectangle implements Shape {
    width
    height

    constructor(w, h) {
        width = w
        height = h
    }

    function area() {
        return width * height
    }
}

class Square implements Shape {
    side

    constructor(s) {
        side = s
    }

    function area() {
        return side * side
    }
}

// Both work correctly with Shape interface
```

## More Examples

### Bad: Bird That Can't Fly

```pseudocode
class Bird {
    function fly() {
        // Move through air
        position.y += altitude
    }
}

class Penguin extends Bird {
    function fly() {
        throw CannotFlyException  // Violates LSP!
    }
}

// Code expecting Bird breaks
function migrateBirds(birds: List<Bird>) {
    for bird in birds {
        bird.fly()  // Crashes on Penguin!
    }
}
```

### Good: Proper Abstraction

```pseudocode
interface Bird {
    function eat()
    function sleep()
}

interface FlyingBird extends Bird {
    function fly()
}

class Sparrow implements FlyingBird {
    function fly() { /* ... */ }
    function eat() { /* ... */ }
    function sleep() { /* ... */ }
}

class Penguin implements Bird {
    function eat() { /* ... */ }
    function sleep() { /* ... */ }
    function swim() { /* ... */ }  // Penguin-specific
}

// Now we can safely migrate flying birds
function migrateBirds(birds: List<FlyingBird>) {
    for bird in birds {
        bird.fly()  // All birds here CAN fly
    }
}
```

### Bad: Read-Only Collection Violation

```pseudocode
class Collection {
    function add(item) {
        items.append(item)
    }

    function remove(item) {
        items.delete(item)
    }
}

class ReadOnlyCollection extends Collection {
    function add(item) {
        throw UnsupportedOperationException  // LSP violation!
    }

    function remove(item) {
        throw UnsupportedOperationException  // LSP violation!
    }
}
```

### Good: Proper Interface Hierarchy

```pseudocode
interface Readable {
    function get(index)
    function size()
    function iterate()
}

interface Writable {
    function add(item)
    function remove(item)
}

interface Collection extends Readable, Writable { }

class ArrayList implements Collection {
    // Implements all methods
}

class ReadOnlyList implements Readable {
    // Only implements read methods
}
```

## LSP Rules

### 1. Signature Rules
- Subtype methods must accept same or broader parameter types (contravariance)
- Subtype methods must return same or narrower types (covariance)
- Subtype must not throw new exceptions (except subtypes of parent's exceptions)

### 2. Behavioral Rules
- Preconditions cannot be strengthened in subtype
- Postconditions cannot be weakened in subtype
- Invariants must be maintained

```pseudocode
// Parent's contract
class Account {
    balance

    // Precondition: amount > 0
    // Postcondition: balance decreased by amount
    function withdraw(amount) {
        if amount <= 0 {
            throw InvalidAmount
        }
        balance -= amount
    }
}

// Bad subclass - strengthens precondition
class PremiumAccount extends Account {
    function withdraw(amount) {
        if amount < 100 {  // Stricter! Violates LSP
            throw MinimumWithdrawal
        }
        balance -= amount
    }
}

// Good subclass - maintains contract
class RewardsAccount extends Account {
    function withdraw(amount) {
        if amount <= 0 {
            throw InvalidAmount
        }
        balance -= amount
        addRewardPoints(amount)  // Additional behavior OK
    }
}
```

## Design by Contract

LSP is closely related to Design by Contract:

```pseudocode
class Stack {
    // Invariant: size >= 0

    // Precondition: true (always allowed)
    // Postcondition: size increased by 1, top() returns item
    function push(item) { }

    // Precondition: size > 0
    // Postcondition: size decreased by 1, returns previous top
    function pop() { }
}

// Any subclass must honor these contracts
class BoundedStack extends Stack {
    maxSize

    // Can ADD preconditions to new methods
    function setMaxSize(size) {
        require(size > 0)  // New method, new precondition OK
    }

    // But CANNOT strengthen existing preconditions
    function push(item) {
        // Cannot require size < maxSize as precondition!
        // Instead, document as additional behavior
        if size >= maxSize {
            // Handle overflow (expand, throw, etc.)
        }
        super.push(item)
    }
}
```

## Testing LSP

Write tests that use the base type and run them against all subtypes:

```pseudocode
function testStack(stack: Stack) {
    assert(stack.isEmpty())

    stack.push("A")
    assert(not stack.isEmpty())
    assert(stack.top() == "A")

    stack.push("B")
    assert(stack.pop() == "B")
    assert(stack.pop() == "A")
    assert(stack.isEmpty())
}

// Run same test on all implementations
testStack(new ArrayStack())
testStack(new LinkedStack())
testStack(new BoundedStack(10))
// All should pass!
```

## Benefits

- **Reliable polymorphism**: Subclasses work wherever parent is expected
- **Reusable code**: Functions work with any subtype
- **Safer refactoring**: Replace implementations confidently
- **Clear contracts**: Behavior is predictable

## Related Principles

- **Open/Closed**: LSP enables safe extension
- **Interface Segregation**: Small interfaces are easier to implement correctly
- **Design by Contract**: Formal specification of LSP

---

*Based on concepts from "Clean Code" and "Agile Software Development" by Robert C. Martin, and Barbara Liskov's original paper.*
