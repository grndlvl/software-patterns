# Environment Model: Understanding Assignment and State

## The Problem with Substitution Model

The **substitution model** of evaluation works fine for **pure functions** where we evaluate an expression by replacing names with their values. However, it breaks down when we introduce **assignment** (`set!` in Scheme-like languages, or variable mutation in general).

### Why Substitution Fails

```pseudocode
// Pure function (substitution model works)
add(a, b) = a + b

evaluate: add(3, 4)
substitute: 3 + 4
result: 7
```

But with assignment:

```pseudocode
// With mutable state (substitution model fails)
balance = 100
balance = balance - 20
balance = balance + 50

// Which value should substitute for "balance"?
// 100? 80? 130? 
// Answer depends on ORDER of execution and HISTORY
// → Substitution model cannot capture this
```

**Core insight:** Assignment breaks referential transparency. The same name can refer to different values at different times. The substitution model assumes names always refer to the same value - it has no concept of **time** or **state change**.

---

## Environment Model: Frames and Bindings

The **environment model** replaces substitution with a more realistic view:

- **Environment** = A sequence of **frames** that maps names to values
- **Frame** = A table of bindings (variable → value pairs)
- **Binding** = An association between a variable name and its current value
- **Evaluation** happens in an environment context that can change

### Environments as Linked Frames

An environment is a **chain of frames**, where each frame extends the previous:

```
Global Environment
├─ Frame 1: {x: 10, y: 20, f: <procedure>}
│  └─ Frame 2: {x: 5, z: 30}          ← More recent frame
│     └─ Frame 3: {x: 1}              ← Current frame

Search for "x": Frame 3 has it (1)
Search for "y": Not in Frame 3, search up to Frame 1 (20)
Search for "z": Frame 3 and 2 don't have it, stops at Frame 2 (30)
```

Each frame points to an **enclosing environment**:
- The **global environment** is the root (has no enclosing environment)
- Inner frames create a parent-child relationship
- Name lookup searches from innermost frame outward

### Variable Lookup Algorithm

```pseudocode
function lookup(name, environment):
    current_frame = environment.innermost_frame
    
    while current_frame is not null:
        if current_frame.has_binding(name):
            return current_frame.get_binding(name)
        current_frame = current_frame.enclosing_environment
    
    // Not found in any frame
    raise UnboundVariable(name)
```

### Variable Assignment

```pseudocode
function assign(name, value, environment):
    current_frame = environment.innermost_frame
    
    while current_frame is not null:
        if current_frame.has_binding(name):
            current_frame.set_binding(name, value)  // Update existing binding
            return
        current_frame = current_frame.enclosing_environment
    
    // Not found - create new binding in innermost frame
    // (or raise error in strict languages)
    current_frame.add_binding(name, value)
```

---

## Procedure Application Creates New Frame

When a **procedure is called**, the evaluator:

1. Creates a **new frame** extending the procedure's definition environment
2. Binds **formal parameters** to **actual arguments** in this new frame
3. Evaluates the **procedure body** in this new environment
4. Discards the frame when the procedure returns

```pseudocode
procedure apply(procedure, arguments, environment):
    // Procedure object contains:
    // - formal_parameters: list of parameter names
    // - body: procedure code
    // - definition_environment: where procedure was defined
    
    // 1. Create new frame extending definition environment
    new_frame = Frame(enclosing = procedure.definition_environment)
    
    // 2. Bind parameters to arguments
    for i = 0 to length(formal_parameters) - 1:
        param = procedure.formal_parameters[i]
        arg = arguments[i]
        new_frame.add_binding(param, arg)
    
    // 3. Create new environment with this frame
    new_environment = Environment(innermost_frame = new_frame)
    
    // 4. Evaluate body in new environment
    result = evaluate(procedure.body, new_environment)
    
    // 5. Frame is discarded (garbage collected)
    return result
```

### Example: Procedure Call Trace

```pseudocode
// Global environment
balance = 1000
tax_rate = 0.08

function calculate_tax(amount):
    return amount * tax_rate

// Call
result = calculate_tax(100)
```

**Execution trace:**

```
1. Lookup "calculate_tax" in global environment → <procedure>
2. Lookup "amount" to evaluate argument: 100
3. Create new frame:
   {amount: 100, enclosing: global_environment}
4. Evaluate body in new frame:
   - Lookup "amount" → 100 (in new frame)
   - Lookup "tax_rate" → 0.08 (in enclosing global frame)
   - Compute: 100 * 0.08 = 8
5. Return 8, discard frame
```

---

## Closures: Capturing the Definition Environment

A **closure** is a procedure that "remembers" the environment in which it was defined. This is the key to higher-order functions.

```pseudocode
function make_multiplier(n):
    function multiplier(x):
        return x * n
    return multiplier

// Define procedures
times_3 = make_multiplier(3)
times_5 = make_multiplier(5)

// Each closure has different environment
result_1 = times_3(4)  // Returns 12
result_2 = times_5(4)  // Returns 20
```

**Environment diagram:**

```
Global Environment
├─ Frame 1: {n: 3, multiplier: <procedure>}
│  └─ Closure times_3 remembers this frame
│
├─ Frame 2: {n: 5, multiplier: <procedure>}
│  └─ Closure times_5 remembers this frame

When times_3(4) executes:
  - New frame: {x: 4, enclosing: Frame 1}
  - Lookup "n" → Found in Frame 1 → 3
  - Return 4 * 3 = 12

When times_5(4) executes:
  - New frame: {x: 4, enclosing: Frame 2}
  - Lookup "n" → Found in Frame 2 → 5
  - Return 4 * 5 = 20
```

**Key insight:** Each call to `make_multiplier` creates a new frame, and each returned closure captures its definition environment. This allows multiple instances to maintain independent state.

---

## Pseudocode: Frame Creation and Lookup

### Complete Frame System

```pseudocode
class Binding:
    constructor(name, value):
        this.name = name
        this.value = value

class Frame:
    constructor(enclosing = null):
        this.bindings = {}  // Dictionary of name → Binding
        this.enclosing = enclosing
    
    function add_binding(name, value):
        this.bindings[name] = Binding(name, value)
    
    function has_binding(name):
        return name in this.bindings
    
    function get_binding(name):
        if this.has_binding(name):
            return this.bindings[name].value
        raise UnboundVariable(name)
    
    function set_binding(name, value):
        if this.has_binding(name):
            this.bindings[name].value = value
        else:
            raise UnboundVariable(name)

class Environment:
    constructor(innermost_frame):
        this.innermost_frame = innermost_frame
    
    function lookup(name):
        current = this.innermost_frame
        while current is not null:
            if current.has_binding(name):
                return current.get_binding(name)
            current = current.enclosing
        raise UnboundVariable(name)
    
    function assign(name, value):
        current = this.innermost_frame
        while current is not null:
            if current.has_binding(name):
                current.set_binding(name, value)
                return
            current = current.enclosing
        // If not found, add to innermost frame
        this.innermost_frame.add_binding(name, value)

class Procedure:
    constructor(parameters, body, definition_environment):
        this.parameters = parameters
        this.body = body
        this.definition_environment = definition_environment

function apply_procedure(procedure, arguments):
    // Create new frame extending definition environment
    new_frame = Frame(enclosing = procedure.definition_environment.innermost_frame)
    
    // Bind parameters to arguments
    for i = 0 to length(procedure.parameters) - 1:
        new_frame.add_binding(procedure.parameters[i], arguments[i])
    
    // Create new environment
    new_env = Environment(new_frame)
    
    // Evaluate body
    return evaluate(procedure.body, new_env)
```

---

## Environment Diagrams

An **environment diagram** is a visual representation of the environment structure during execution. Rectangles represent frames, with arrows showing enclosing environment relationships.

### Diagram Components

```
┌─────────────────────────────────────┐
│     Global Environment              │
├─────────────────────────────────────┤
│ x: 5                                │
│ y: 10                               │
│ f: ──────────────────────────┐      │
│ g: ──────────────────────────┤      │
└─────────────────────────────────────┘
                ▲
                │ enclosing
        ┌───────┴──────────────┐
        │   Frame 1 (f call)   │
        ├──────────────────────┤
        │ x: 3                 │
        │ z: 7                 │
        └──────────────────────┘
                ▲
                │ enclosing
        ┌───────┴──────────────┐
        │  Frame 2 (nested)    │
        ├──────────────────────┤
        │ a: 99                │
        └──────────────────────┘
```

**Reading the diagram:**
- Each rectangle is a frame
- Arrows point to enclosing environments
- Name lookup follows arrows up the chain
- When procedures return, their frames disappear

### Closure Diagram Example

```
┌──────────────────────────────┐
│  Global Environment          │
├──────────────────────────────┤
│ make_counter: <procedure>    │
│ c1: ─────────────────┐       │
│ c2: ──────────┐      │       │
└──────────────────────────────┘
          ▲
          │
          │ (c1 remembers)
      ┌───┴────────────────┐
      │  Frame A           │
      ├────────────────────┤
      │ count: 0           │
      │ increment: <proc>  │
      └────────────────────┘
          ▲
          │
          │ (c2 remembers)
      ┌───┴────────────────┐
      │  Frame B           │
      ├────────────────────┤
      │ count: 0           │
      │ increment: <proc>  │
      └────────────────────┘

// When c1() is called:
// - New frame for call, enclosing Frame A
// - "count" lookup finds Frame A.count
```

---

## Evaluation Rules with Environments

### Core Evaluation Rule

**To evaluate an expression E in environment Env:**

1. **Literals** (numbers, strings, booleans):
   - Return the literal value directly

2. **Names** (variables):
   - Look up the name in Env using variable lookup algorithm
   - Return the bound value

3. **Special forms** (if, define, set!, etc.):
   - Evaluate according to special form rules

4. **Procedure application** (function call):
   - Evaluate the operator in Env
   - Evaluate each argument in Env
   - Apply the resulting procedure to the argument values
   - Application creates new frame with bindings

### Assignment Evaluation

```pseudocode
// Pseudocode for set! (or assignment)
function evaluate_assignment(name, value_expr, environment):
    value = evaluate(value_expr, environment)
    environment.assign(name, value)  // Update or create binding
    return value
```

**Key difference from pure substitution:**
- The name gets a new value in the environment
- Previous bindings can be overwritten
- Later lookups see the updated value

### Define Evaluation

```pseudocode
function evaluate_define(name, value_expr, environment):
    value = evaluate(value_expr, environment)
    environment.innermost_frame.add_binding(name, value)
    return UNDEFINED  // Define returns no meaningful value
```

**Key difference from assignment:**
- Define creates new binding in **current frame** (usually global)
- Set! updates binding in **existing frame** (searches up chain)

---

## Summary Table: Substitution vs. Environment Model

| Aspect | Substitution Model | Environment Model |
|--------|-------------------|-------------------|
| **Name resolution** | Replace with value | Look up in environment chain |
| **Assignment** | Impossible | Updates frame binding |
| **State** | No concept of state | State = current frame values |
| **Time** | No notion of time | Execution creates frame history |
| **Procedure call** | Simple substitution | Creates new frame, binds parameters |
| **Closure** | No closure concept | Procedure remembers definition environment |
| **Mutable state** | Cannot handle | Natural representation |
| **Variable scope** | Textual (lexical) | Dynamic (environment chain) |
| **Implementation** | Direct string replacement | Frame/environment data structure |

---

## Key Principles

1. **Frame = Scope**: Each frame represents a scope where variables are defined
2. **Environment Chain = Scope Chain**: Looking up names follows parent scopes
3. **Procedure Definition = Environment Capture**: When defined, procedures capture their environment
4. **Procedure Call = New Frame**: Each call creates a fresh frame for parameters
5. **Assignment = Mutation**: Set! updates the binding in the frame where it exists
6. **Lookup Precedes Assignment**: You can only set a variable if it already exists somewhere in the chain

---

## Practical Implications

### Why Order Matters

```pseudocode
x = 10
function f():
    return x

x = 20
result = f()  // Returns 20, not 10

// Environment model explains this:
// - f was defined when x = 10
// - f captures the global environment
// - When f is called, it looks up x
// - x's binding in global environment is now 20
```

### Why Closures Work

```pseudocode
function create_account(initial):
    balance = initial
    
    function withdraw(amount):
        balance = balance - amount
        return balance
    
    return withdraw

account1 = create_account(100)
account2 = create_account(200)

account1(10)  // Returns 90 (account1's balance)
account2(10)  // Returns 190 (account2's balance)
```

Each closure has its own frame with independent `balance` binding.

### Variable Shadowing

```pseudocode
x = 5          // Global frame

function f(x):  // Parameter x shadows global x
    return x

function g():
    x = 3       // Local x shadows global x
    return x

f(10)  // Returns 10 (parameter binding)
g()    // Returns 3 (local binding)
x      // Still 5 (global binding unchanged)
```

---

*Based on concepts from "Structure and Interpretation of Computer Programs" by Abelson and Sussman.*
