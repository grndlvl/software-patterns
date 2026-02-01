# Data Abstraction

## Definition

Data abstraction is the principle of **separating how data is used from how it is represented**. It allows you to work with data through a well-defined interface (constructors and selectors) without needing to know or depend on the underlying implementation.

**Key Insight:** What matters is not *what* the data is, but *what operations it supports*.

### The Contract

A data abstraction is defined by:
1. **Constructor(s)**: Functions that create the abstraction
2. **Selector(s)**: Functions that extract information from it
3. **Operations**: Functions that manipulate it
4. **Invariants**: Properties the abstraction maintains

Users of the abstraction interact only through these, never directly with the representation.

---

## Constructors and Selectors

A typical data abstraction uses two types of functions:

### Constructor
Creates an instance of the abstraction, bundling multiple pieces of data together.

```
// Pseudocode
function make-point(x, y)
    return [x, y]
```

### Selectors
Extract individual pieces from an abstraction.

```
// Pseudocode
function x-coord(point)
    return point[0]

function y-coord(point)
    return point[1]
```

### Using the Abstraction

```
// Users never access point[0] or point[1] directly
let p = make-point(3, 4)
let x = x-coord(p)    // 3
let y = y-coord(p)    // 4
```

**Why this matters:** If the implementation of `make-point` changes (say, from array to object), *only* the constructor and selectors change. All client code continues working.

---

## Abstraction Barriers

Abstraction barriers create **layers** in your program. Each layer depends only on the layers below it through well-defined interfaces.

```
┌─────────────────────────────────────┐
│  Application Code                   │  ← Uses rational numbers
│  (e.g., add_rationals, multiply...) │
├─────────────────────────────────────┤
│  ABSTRACTION BARRIER                │
├─────────────────────────────────────┤
│  Rational Number Interface          │  ← Uses make_rat, numer, denom
│  (make-rat, numer, denom)           │
├─────────────────────────────────────┤
│  ABSTRACTION BARRIER                │
├─────────────────────────────────────┤
│  Pair Implementation                │  ← Uses make_pair, car, cdr
│  (make-pair, car, cdr)              │
├─────────────────────────────────────┤
│  ABSTRACTION BARRIER                │
├─────────────────────────────────────┤
│  Language Primitives                │  ← Arrays, objects, or functions
│  (array, object, function)          │
└─────────────────────────────────────┘
```

Each layer:
- **Knows about** layers below it
- **Doesn't know about** layers above it
- **Communicates through** defined interfaces only

**Benefit:** You can change a lower layer's implementation without affecting layers above it.

---

## Rational Numbers Example

A rational number is a pair of integers (numerator and denominator) representing a fraction.

### The Abstraction Barrier

Users think of a rational number as a single conceptual unit with two parts:
- A numerator
- A denominator

### Implementation Layer 1: Constructor and Selectors

```
// Constructor: create a rational number
function make-rat(n, d)
    // Store as pair [n, d], reduced to lowest terms
    let g = gcd(n, d)
    return [n / g, d / g]

// Selectors
function numer(rat)
    return rat[0]

function denom(rat)
    return rat[1]
```

### Using Rational Numbers

```
// Application code uses only the interface
let r = make-rat(6, 9)      // Simplified to [2, 3]
let n = numer(r)             // 2
let d = denom(r)             // 3

function add-rat(r1, r2)
    return make-rat(
        numer(r1) * denom(r2) + numer(r2) * denom(r1),
        denom(r1) * denom(r2)
    )

function multiply-rat(r1, r2)
    return make-rat(
        numer(r1) * numer(r2),
        denom(r1) * denom(r2)
    )

let r3 = add-rat(make-rat(1, 2), make-rat(1, 3))
// Result: [5, 6]
```

### Why This Design?

| Decision | Benefit |
|----------|---------|
| Reduce in constructor | All rationals automatically in lowest terms |
| Selectors return parts | Users can build operations on rationals |
| Pairs as foundation | Don't need special rational-number hardware |
| Hidden representation | Can change [n,d] to {num: n, den: d} later |

---

## Data as Procedures

A surprising insight: **data can be represented as procedures**.

Instead of storing a pair as a structure, you can create a function that *remembers* the components:

### Pair as Structure (Conventional)

```
function make-pair(a, b)
    return [a, b]

function car(pair)
    return pair[0]

function cdr(pair)
    return pair[1]
```

### Pair as Procedure (Church Encoding)

```
// Create a pair using a function that remembers a and b
function make-pair(a, b)
    return function(m)
        if m == 0
            return a
        else if m == 1
            return b

// Extract first component by asking the pair for it
function car(pair)
    return pair(0)

// Extract second component by asking the pair for it
function cdr(pair)
    return pair(1)
```

### They Behave Identically

```
// Using structural representation
let p1 = make-pair(3, 5)
car(p1)   // 3
cdr(p1)   // 5

// Using procedural representation
let p2 = make-pair(3, 5)
car(p2)   // 3
cdr(p2)   // 5
```

**Key Insight:** There's no inherent difference between "data" and "procedures". A procedure that carries information *is* data. What matters is the interface contract.

---

## Message Passing

Data can communicate through **message passing**: the object receives a message (as a symbol or string) and responds accordingly.

### Message-Passing Pair

```
function make-pair-message(a, b)
    return function(message)
        if message == "car"
            return a
        else if message == "cdr"
            return b
        else if message == "type"
            return "pair"
        else
            error("Unknown message: " + message)

function car(pair)
    return pair("car")

function cdr(pair)
    return pair("cdr")
```

### Message-Passing Rectangle

```
function make-rectangle(width, height)
    return function(message)
        if message == "area"
            return width * height
        else if message == "perimeter"
            return 2 * (width + height)
        else if message == "width"
            return width
        else if message == "height"
            return height

let rect = make-rectangle(4, 5)
rect("area")       // 20
rect("perimeter")  // 18
rect("width")      // 4
```

**Why Message Passing?**
- Object encapsulates its own behavior
- Adding new operations doesn't require modifying constructor
- Natural fit for multi-method dispatch systems

---

## What Is Data?

In the most abstract sense, **data is anything that fulfills a contract**.

### The Contract

A data abstraction is "data" if:

1. **It bundles information together**
   - Multiple pieces grouped as one unit
   - Could be constructor + selectors, or a procedure, or anything

2. **It supports operations defined by the interface**
   - Users access it through an interface
   - Never directly manipulating internal representation

3. **It maintains invariants**
   - Operations preserve consistent state
   - Users can trust the abstraction

### Examples of "Data"

| What | Constructor | Selectors | Why It's Data |
|------|-------------|-----------|---------------|
| Rational number | `make-rat` | `numer`, `denom` | Bundles numerator + denominator; maintains reduced form |
| Procedure | `make-pair` | `car`, `cdr` | Procedure that remembers information; can be queried |
| Message object | `make-rectangle` | message symbols | Encapsulates state; responds to operations |
| Bank account | `make-account` | `withdraw`, `deposit`, `balance` | Bundles balance; responds to transactions |

**There is no essential difference between "procedures" and "data".**

---

## Pseudocode Examples

### Example 1: Point Abstraction

```
// Layer 1: Constructor and selectors
function make-point(x, y)
    return [x, y]

function x-coord(p)
    return p[0]

function y-coord(p)
    return p[1]

// Layer 2: Operations using the abstraction
function distance(p1, p2)
    let dx = x-coord(p2) - x-coord(p1)
    let dy = y-coord(p2) - y-coord(p1)
    return sqrt(dx * dx + dy * dy)

function midpoint(p1, p2)
    return make-point(
        (x-coord(p1) + x-coord(p2)) / 2,
        (y-coord(p1) + y-coord(p2)) / 2
    )

// Layer 3: Application using operations
let p1 = make-point(0, 0)
let p2 = make-point(3, 4)
let d = distance(p1, p2)        // 5
let m = midpoint(p1, p2)        // [1.5, 2]
```

### Example 2: Stack Abstraction

```
// Constructor: empty stack
function make-stack()
    let items = []
    return function(operation)
        if operation == "push"
            return function(value)
                items.push(value)
        else if operation == "pop"
            return function()
                return items.pop()
        else if operation == "empty?"
            return items.length == 0
        else if operation == "size"
            return items.length

// Selectors and operations
function push(stack, value)
    stack("push")(value)

function pop(stack)
    return stack("pop")()

function is-empty(stack)
    return stack("empty?")

// Usage
let s = make-stack()
push(s, 1)
push(s, 2)
push(s, 3)
let top = pop(s)              // 3
is-empty(s)                   // false
```

### Example 3: Complex Number Abstraction

```
// Rectangular representation
function make-rect(real, imag)
    return [real, imag]

function real-part-rect(z)
    return z[0]

function imag-part-rect(z)
    return z[1]

// Polar representation (same abstraction, different implementation)
function make-polar(mag, angle)
    return [mag, angle]

function real-part-polar(z)
    return z[0] * cos(z[1])

function imag-part-polar(z)
    return z[0] * sin(z[1])

// Generic selectors (choose implementation)
let z = make-rect(3, 4)
real-part(z)   // Uses make-rect? real-part-rect(z)
imag-part(z)   // Uses make-rect? imag-part-rect(z)

// Operations don't care about representation
function add-complex(z1, z2)
    return make-rect(
        real-part(z1) + real-part(z2),
        imag-part(z1) + imag-part(z2)
    )

function magnitude(z)
    return sqrt(real-part(z)² + imag-part(z)²)
```

---

## Benefits Table

| Benefit | Description | Example |
|---------|-------------|---------|
| **Modularity** | Code divides into independent components | Rational arithmetic doesn't know about GCD algorithm |
| **Changeability** | Internals change without affecting users | Switch from `[n,d]` to `{num,den}` transparently |
| **Testing** | Each layer tested independently | Test `make-rat` separately from `add-rat` |
| **Reusability** | Abstraction works in multiple contexts | Pair constructor used for rationals, points, complex numbers |
| **Clarity** | Code expresses intent, not implementation | `add-rat` reads naturally; details hidden |
| **Maintenance** | Bug fixes stay local | Fix GCD algorithm; no other code changes |
| **Composition** | Build complex systems from simple pieces | Stack uses pair; account uses stack |
| **Flexibility** | Multiple implementations behind same interface | Rectangular vs. polar complex numbers |

---

## Summary Table

| Concept | Purpose | Key Rules |
|---------|---------|-----------|
| **Constructor** | Create abstraction | Bundle all needed data; can validate |
| **Selectors** | Extract information | Provide named access, hide structure |
| **Abstraction Barrier** | Separate concerns | Each layer knows only about layer below |
| **Invariant** | Maintain consistency | Constructor/operations preserve properties |
| **Interface** | Define contract | Users depend only on interface, not implementation |
| **Procedural Data** | Alternative representation | Functions can embody data through closure |
| **Message Passing** | Encapsulated behavior | Object responds to named operations |

### The Fundamental Pattern

```
┌─────────────────────────────┐
│  Conceptual Entity (e.g.,   │
│  rational number)           │
└────────────┬────────────────┘
             │ make-rat
             │ numer, denom
             │
             ↓
┌─────────────────────────────┐
│  Implementation Details     │
│  (array, object, function)  │
└─────────────────────────────┘
```

**The user knows the concept and interface. The implementation is interchangeable.**

---

*Based on concepts from "Structure and Interpretation of Computer Programs" by Abelson and Sussman.*
