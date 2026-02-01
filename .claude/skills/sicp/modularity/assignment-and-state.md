# Assignment and State: Modularity Through Mutable State

## Introduction: Why Assignment?

In purely functional programming, we compute values but never change them. However, **real systems maintain state**: bank account balances change, user sessions have current data, simulations progress through time. Assignment (mutation) allows us to model these systems naturally.

### The Tension

- **Without assignment**: Pure functions are easy to reason about but modeling state requires threading state through parameters
- **With assignment**: State is localized and intuitive but reasoning becomes harder

This module explores how **controlled assignment** enables **modularity and encapsulation** despite the costs.

---

## Local State Variables

### Concept: Private Mutable Storage

A **local state variable** is a piece of mutable storage that belongs to an object or closure. Unlike global variables, local state is **encapsulated** - only the object that created it can access it.

```pseudocode
// Without local state (functional approach)
function withdraw_functional(balance, amount):
    if amount > balance:
        return "Insufficient funds"
    return balance - amount

// Problem: Balance not maintained between calls
result1 = withdraw_functional(100, 10)  // Returns 90
result2 = withdraw_functional(90, 20)   // Returns 70
// But the original balance (100) is not updated

// With local state (imperative approach)
function make_account(initial_balance):
    balance = initial_balance
    
    function withdraw(amount):
        if amount > balance:
            return "Insufficient funds"
        balance = balance - amount
        return balance
    
    function deposit(amount):
        balance = balance + amount
        return balance
    
    function get_balance():
        return balance
    
    return {
        withdraw: withdraw,
        deposit: deposit,
        balance: get_balance
    }

// Usage
account = make_account(100)
account.withdraw(10)      // balance becomes 90, returns 90
account.withdraw(20)      // balance becomes 70, returns 70
account.deposit(50)       // balance becomes 120, returns 120
account.balance()         // returns 120
```

### Key Properties

1. **Encapsulation**: Only the closure can modify `balance`
2. **Persistence**: State survives between function calls
3. **Independence**: Each account has its own balance

```pseudocode
account1 = make_account(100)
account2 = make_account(200)

account1.withdraw(10)  // account1.balance = 90
account2.withdraw(10)  // account2.balance = 190
// account1 and account2 have independent state
```

---

## Benefits of Assignment

### 1. Modularity: Organizing Complex State

**Problem without assignment:**

```pseudocode
// Functional simulation requires threading state through all functions
function simulate_step(particles, forces, dt):
    new_particles = []
    for particle in particles:
        acceleration = compute_acceleration(particle, forces)
        velocity = particle.velocity + acceleration * dt
        position = particle.position + velocity * dt
        new_particles.push({
            position: position,
            velocity: velocity,
            acceleration: acceleration
        })
    return new_particles

// Usage - state threading is explicit but verbose
state1 = initial_state
state2 = simulate_step(state1, forces, 0.01)
state3 = simulate_step(state2, forces, 0.01)
// ... repeat
state100 = simulate_step(state99, forces, 0.01)
```

**Solution with assignment:**

```pseudocode
function make_particle(position, velocity):
    acceleration = 0.0
    
    function update(force, dt):
        acceleration = force / mass
        velocity = velocity + acceleration * dt
        position = position + velocity * dt
    
    function get_position():
        return position
    
    return {update: update, position: get_position}

// Usage - each particle manages its own state
particles = [make_particle(...), make_particle(...), ...]

function simulate_step(particles, forces, dt):
    for i = 0 to length(particles) - 1:
        particles[i].update(forces[i], dt)

simulate_step(particles, forces, 0.01)
simulate_step(particles, forces, 0.01)
// ... repeat
simulate_step(particles, forces, 0.01)
```

**Benefit:** Each object encapsulates its own state. The simulator doesn't need to know particle implementation details.

### 2. Encapsulation: Hiding Implementation

```pseudocode
// Client code doesn't know how balance is stored
function make_account(initial):
    balance = initial
    transactions = []  // Internal implementation detail
    
    function withdraw(amount):
        balance = balance - amount
        transactions.push({type: "withdrawal", amount: amount})
        return balance
    
    return {withdraw: withdraw}

// Client can only call withdraw, can't access balance or transactions directly
account = make_account(100)
account.withdraw(10)
// account.balance would cause error (not exposed)
// account.transactions would cause error (not exposed)
```

### 3. Natural Expression of Real-World Concepts

```pseudocode
// Bank account: Balance changes over time
function make_bank_account(initial_balance):
    balance = initial_balance
    
    function update(amount):
        balance = balance + amount
        return balance
    
    return {deposit: (amt) => update(amt), withdraw: (amt) => update(-amt)}

// Simulation: Particle moves
function make_moving_particle(x, y):
    position = {x: x, y: y}
    velocity = {x: 0, y: 0}
    
    function apply_force(fx, fy):
        acceleration_x = fx / mass
        acceleration_y = fy / mass
        velocity.x = velocity.x + acceleration_x * dt
        velocity.y = velocity.y + acceleration_y * dt
        position.x = position.x + velocity.x * dt
        position.y = position.y + velocity.y * dt
    
    return {apply_force: apply_force, get_position: () => position}

// UI Component: Widget state changes
function make_text_input():
    text = ""
    focused = false
    
    function on_key(char):
        text = text + char
        return text
    
    function on_focus():
        focused = true
    
    function on_blur():
        focused = false
    
    return {on_key: on_key, on_focus: on_focus, on_blur: on_blur, get_text: () => text}
```

---

## Costs of Assignment

### 1. Loss of Referential Transparency

**Referential transparency**: An expression always evaluates to the same value.

```pseudocode
// Pure function (referential transparent)
double(5) == double(5)  // Always true

// With state (not referential transparent)
function make_counter():
    count = 0
    function next():
        count = count + 1
        return count
    return next

counter = make_counter()
counter() == counter()  // FALSE: returns 1, then 2

// You cannot replace counter() with a constant
x = counter()           // x = 1
y = counter()           // y = 2
x == y                  // FALSE

// But in pure code:
x = double(5)           // x = 10
y = double(5)           // y = 10
x == y                  // TRUE (always)
```

**Consequence:** You can't use **algebraic reasoning** anymore.

### 2. Reasoning Becomes Harder

```pseudocode
// Pure code - easy to reason about
function compute(a, b, c):
    x = a + b
    y = x * 2
    z = y - c
    return z

// The value of x is always (a + b)
// The value of y is always (a + b) * 2
// You can substitute without worry

// With state - harder to reason about
function make_account(initial):
    balance = initial
    
    function withdraw(amount):
        balance = balance - amount
        return balance
    
    return withdraw

acc = make_account(100)
acc(10)      // balance = 90, returns 90
acc(20)      // balance = 70, returns 70
acc(5)       // balance = 65, returns 65

// What is the balance? Depends on:
// - Initial value (100)
// - History of operations (withdrew 10, 20, 5)
// - ORDER of operations

// Swap operations
acc = make_account(100)
acc(20)      // returns 80
acc(10)      // returns 70 (different from 65!)
acc(5)
```

**Why harder?**
- Final value depends on **execution order**
- Must track **history of mutations**
- Hard to predict value at arbitrary point in time
- Debugging requires understanding **entire execution trace**

### 3. Concurrency Becomes Complex

```pseudocode
// Pure code - safe for concurrent execution
function square(n):
    return n * n

// Multiple threads can call square(5) simultaneously
// Each always gets 25, no conflicts

// With mutable state - concurrency hazards
function make_account(initial):
    balance = initial
    
    function withdraw(amount):
        if amount <= balance:
            balance = balance - amount  // ← Shared mutable state
            return true
        return false
    
    return withdraw

account = make_account(100)

// Thread A: withdraw(60)
// Thread B: withdraw(60)
// Both check balance (100) - both pass check
// Both decrement balance
// Result: balance = -20 (wrong!)
// Should only allow one withdrawal

// Solution: Add locks (but now code is more complex)
function make_account(initial):
    balance = initial
    lock = create_lock()
    
    function withdraw(amount):
        acquire_lock(lock)
        try:
            if amount <= balance:
                balance = balance - amount
                return true
            return false
        finally:
            release_lock(lock)
    
    return withdraw
```

### 4. Testing Becomes Harder

```pseudocode
// Pure function - easy to test
function add(a, b):
    return a + b

test_add():
    assert add(2, 3) == 5
    assert add(-1, 1) == 0
    assert add(0, 0) == 0
    // Each test is independent

// Stateful object - harder to test
function make_account(initial):
    balance = initial
    
    function deposit(amount):
        balance = balance + amount
        return balance
    
    return deposit

test_account():
    acc = make_account(100)
    
    // First test
    assert acc(10) == 110  // Changes internal state
    
    // Second test - affected by first test!
    assert acc(10) == 120  // Can't test in isolation
    
    // Must reset state between tests
    acc = make_account(100)  // Reset needed
    
    // Or create new instance for each test
    assert acc2(10) == 110   // But this duplicates setup
```

### 5. Equality Becomes Ambiguous

```pseudocode
// Pure values - equality is clear
5 == 5                // True
double(5) == double(5)  // True

// Objects with mutable state - what does equality mean?
function make_account(initial):
    balance = initial
    function get_balance():
        return balance
    function withdraw(amount):
        balance = balance - amount
    return {get_balance: get_balance, withdraw: withdraw}

acc1 = make_account(100)
acc2 = make_account(100)

// Are they equal?
acc1 == acc2  // Depends on definition!

// Same identity?
acc1 is acc1  // True
acc1 is acc2  // False

// Same current value?
acc1.get_balance() == acc2.get_balance()  // True
acc1.withdraw(10)
acc1.get_balance() == acc2.get_balance()  // False

// Same historical behavior?
// acc1 and acc2 start the same but diverge with mutations
// There's no single answer to "are they equal?"
```

---

## Bank Account Example: Complete Implementation

### Full Bank Account Object

```pseudocode
function make_bank_account(initial_balance):
    balance = initial_balance
    transactions = []  // History of all transactions
    
    function record_transaction(type, amount):
        transactions.push({
            type: type,
            amount: amount,
            timestamp: current_time(),
            balance_after: balance
        })
    
    function withdraw(amount):
        if amount > balance:
            return {success: false, message: "Insufficient funds"}
        
        balance = balance - amount
        record_transaction("withdrawal", amount)
        
        return {success: true, new_balance: balance}
    
    function deposit(amount):
        if amount <= 0:
            return {success: false, message: "Deposit must be positive"}
        
        balance = balance + amount
        record_transaction("deposit", amount)
        
        return {success: true, new_balance: balance}
    
    function get_balance():
        return balance
    
    function get_transactions():
        return transactions  // Could be immutable copy
    
    function transfer(amount, other_account):
        result = this.withdraw(amount)
        if result.success:
            other_result = other_account.deposit(amount)
            if not other_result.success:
                // Rollback - this is complex!
                balance = balance + amount
                transactions.pop()
                return {success: false, message: "Transfer failed"}
        return result
    
    return {
        withdraw: withdraw,
        deposit: deposit,
        get_balance: get_balance,
        get_transactions: get_transactions,
        transfer: transfer
    }

// Usage
account1 = make_bank_account(1000)
account2 = make_bank_account(500)

account1.withdraw(100)        // {success: true, new_balance: 900}
account1.deposit(50)          // {success: true, new_balance: 950}
account1.transfer(100, account2)  // Transfer 100 from account1 to account2

account1.get_balance()        // 850
account2.get_balance()        // 600
account1.get_transactions()   // Array of transaction records
```

### Problems This Reveals

1. **Lost deposits**: What if transfer fails partway through?
2. **Concurrent transfers**: Two threads transfer simultaneously
3. **Audit trail**: Transaction history can't be undone
4. **Distributed accounts**: How to transfer between accounts at different locations?

These problems motivate **alternative approaches** (ACID transactions, event sourcing, etc.).

---

## Sameness and Change: Identity Problems

### The Identity Crisis

```pseudocode
function make_account(initial):
    balance = initial
    function get_balance():
        return balance
    function withdraw(amount):
        balance = balance - amount
        return balance
    return {get_balance: get_balance, withdraw: withdraw}

account = make_account(100)

// Question: Is this the same account?
account1 = account
account2 = account
account1 is account2  // True (same object)

// What about this?
account1 = make_account(100)
account2 = make_account(100)
account1 is account2  // False (different objects)

// But they're indistinguishable at creation time:
account1.get_balance() == account2.get_balance()  // True (both 100)

// After operations they diverge:
account1.withdraw(10)
account1.get_balance() == account2.get_balance()  // False (90 vs 100)

// So when did they become different?
// Answer: At the moment they were created (different identities)
// But we only notice the difference after mutation
```

### Immutable vs. Mutable View

```pseudocode
// Functional (immutable) view
// Each "state" is a separate value
state0 = {balance: 100}
state1 = {balance: 90}   // New state, original unchanged
state2 = {balance: 70}   // Another new state

state0.balance == 100  // True - state0 never changed

// Imperative (mutable) view
// One entity changes over time
account = make_account(100)
// account "is" at balance 100

account.withdraw(10)
// account "is" at balance 90 (same object, different state)

// Philosophical question: Is it the same account?
// - Identity perspective: Yes (same object)
// - Value perspective: No (different balance)
// - Process perspective: Yes (same history)
```

### Defining Sameness

Without mutation, "sameness" is clear:
```pseudocode
// Pure values
5 == 5             // True
double(5) == double(5)  // True
```

With mutation, we need a definition:

1. **Object identity** (same memory location):
   ```pseudocode
   account1 = make_account(100)
   account2 = account1
   account1 is account2  // True (same object)
   ```

2. **Structural equality** (same current values):
   ```pseudocode
   account1 = make_account(100)
   account2 = make_account(100)
   account1.get_balance() == account2.get_balance()  // True (both 100)
   ```

3. **Behavioral equivalence** (same responses to stimuli):
   ```pseudocode
   // If we apply the same sequence of operations,
   // do we get the same results?
   account1 = make_account(100)
   account2 = make_account(100)
   
   account1.withdraw(10)
   account2.withdraw(10)
   
   account1.get_balance() == account2.get_balance()  // True (both 90)
   // But this only works if we keep them in sync!
   ```

**The key insight:** In the presence of mutation, **object identity becomes essential**. We must distinguish between "the same object" and "an object with the same value."

---

## Functional vs. Imperative: Comparison

### Pure Functional Approach

```pseudocode
// Withdraw as a pure function
function withdraw(balance, amount):
    if amount > balance:
        return {success: false, balance: balance}
    return {success: true, balance: balance - amount}

// Usage - threading state through calls
initial_balance = 100

result1 = withdraw(initial_balance, 10)
balance_after_1 = result1.balance  // 90

result2 = withdraw(balance_after_1, 20)
balance_after_2 = result2.balance  // 70

result3 = withdraw(balance_after_2, 5)
balance_after_3 = result3.balance  // 65

// Final state
final_balance = balance_after_3  // 65
```

**Characteristics:**
- Functions are pure (no side effects)
- Easy to test (same input always gives same output)
- Easy to reason about (no temporal reasoning needed)
- State threading is verbose
- Immutable - full history available
- Hard to model stateful systems

### Imperative Approach

```pseudocode
// Withdraw as a method that mutates internal state
function make_account(initial):
    balance = initial
    
    function withdraw(amount):
        if amount > balance:
            return {success: false}
        balance = balance - amount  // ← Mutation
        return {success: true, balance: balance}
    
    return {withdraw: withdraw, get_balance: () => balance}

// Usage - object maintains state
account = make_account(100)

account.withdraw(10)   // Internal state: 90
account.withdraw(20)   // Internal state: 70
account.withdraw(5)    // Internal state: 65

final_balance = account.get_balance()  // 65
```

**Characteristics:**
- Methods have side effects (mutations)
- Harder to test (need state management)
- Harder to reason about (temporal reasoning needed)
- State is implicit and encapsulated
- Mutable - history is overwritten
- Natural for stateful systems

### Trade-Off Comparison

| Aspect | Functional | Imperative |
|--------|-----------|-----------|
| **Code clarity** | Verbose state threading | Concise, natural |
| **Testability** | Pure functions easy to test | Requires state setup/reset |
| **Reasoning** | Substitution model | Must track execution history |
| **State management** | Explicit (threading) | Implicit (encapsulation) |
| **Immutability** | Complete | None |
| **History access** | Full state history available | Lost (overwritten) |
| **Concurrency** | No locks needed | Locks needed |
| **Performance** | State copying overhead | Efficient mutation |
| **Referential transparency** | Preserved | Lost |
| **Debugging** | Trace any point by replaying | Must inspect current state |

---

## When to Use State (Assignment)

### Use Assignment When:

1. **Encapsulation is the goal**
   ```pseudocode
   // Bad: Global state
   global_balance = 100
   
   function withdraw(amount):
       global global_balance
       global_balance = global_balance - amount
   
   // Good: Encapsulated state
   account = make_account(100)
   account.withdraw(amount)  // Local state, not global
   ```

2. **Modeling natural state transitions**
   ```pseudocode
   // Natural: State changing over time
   player = make_game_character(health=100, ammo=30)
   player.take_damage(10)     // health becomes 90
   player.fire_weapon()       // ammo becomes 29
   
   // Awkward: Threading state through functions
   state = {health: 100, ammo: 30}
   state = take_damage(state, 10)
   state = fire_weapon(state)
   ```

3. **Hiding complex implementation**
   ```pseudocode
   // Client doesn't need to know about internal caching
   function make_expensive_computation():
       cache = {}
       
       function compute(input):
           if input not in cache:
               cache[input] = expensive_operation(input)  // ← Mutation
           return cache[input]
       
       return compute
   ```

4. **Performance optimization**
   ```pseudocode
   // Functional: Create new array each time
   function remove_item_functional(array, item):
       result = []
       for each elem in array:
           if elem != item:
               result.push(elem)
       return result  // O(n) space
   
   // Imperative: Mutate in place
   function remove_item_imperative(array, item):
       i = 0
       while i < length(array):
           if array[i] == item:
               array.remove_at(i)  // ← Mutation
           else:
               i = i + 1
       // O(1) space
   ```

### Avoid Assignment When:

1. **Values should be immutable**
   ```pseudocode
   // Bad: Mutable configuration
   config = {port: 3000, host: "localhost"}
   config.port = 8000  // Surprise: affects other code using config
   
   // Good: Immutable values
   config1 = {port: 3000, host: "localhost"}
   config2 = create_new_config(config1, {port: 8000})
   ```

2. **Sharing is important**
   ```pseudocode
   // Bad: Can't trust this
   function process(account):
       account.balance = 0  // Mutation!
       return account
   
   my_account = make_account(1000)
   process(my_account)  // my_account.balance is now 0!
   
   // Good: Clear that state is preserved
   function get_balance(account):
       return account.get_balance()  // Just reading
   ```

3. **The system is reactive**
   ```pseudocode
   // Bad: Hidden state changes
   function update():
       player.position = player.position + player.velocity  // ← Hidden
       render(player)  // Caller doesn't know player changed
   
   // Better: Explicit return of new state
   function update(player):
       new_player = create_copy(player)
       new_player.position = player.position + player.velocity
       return new_player
   
   player = update(player)  // Explicit that player changed
   ```

---

## Pseudocode Examples

### Example 1: Simple Counter

```pseudocode
function make_counter():
    count = 0
    
    function increment():
        count = count + 1
        return count
    
    function decrement():
        count = count - 1
        return count
    
    function get_value():
        return count
    
    function reset():
        count = 0
        return count
    
    return {
        increment: increment,
        decrement: decrement,
        value: get_value,
        reset: reset
    }

// Usage
counter = make_counter()
counter.increment()      // count = 1, returns 1
counter.increment()      // count = 2, returns 2
counter.decrement()      // count = 1, returns 1
counter.value()          // returns 1
counter.reset()          // count = 0, returns 0

// Multiple independent counters
c1 = make_counter()
c2 = make_counter()
c1.increment()           // c1 is 1
c2.increment()
c2.increment()           // c2 is 2
// c1 and c2 have independent state
```

### Example 2: Queue with Local State

```pseudocode
function make_queue():
    front = null    // Head of queue (linked list)
    rear = null     // Tail of queue
    size = 0
    
    function enqueue(value):
        node = {data: value, next: null}
        
        if rear is null:
            front = node  // Queue was empty
            rear = node
        else:
            rear.next = node
            rear = node
        
        size = size + 1
        return true
    
    function dequeue():
        if front is null:
            return {success: false, value: null}
        
        value = front.data
        front = front.next
        size = size - 1
        
        if front is null:  // Queue is now empty
            rear = null
        
        return {success: true, value: value}
    
    function peek():
        if front is null:
            return null
        return front.data
    
    function get_size():
        return size
    
    function is_empty():
        return size == 0
    
    return {
        enqueue: enqueue,
        dequeue: dequeue,
        peek: peek,
        size: get_size,
        is_empty: is_empty
    }

// Usage
queue = make_queue()
queue.enqueue(1)
queue.enqueue(2)
queue.enqueue(3)

queue.peek()           // returns 1 (first in)
queue.dequeue()        // returns {success: true, value: 1}
queue.dequeue()        // returns {success: true, value: 2}
queue.get_size()       // returns 1
```

### Example 3: Password-Protected Account

```pseudocode
function make_secure_account(initial_balance, password):
    balance = initial_balance
    transaction_limit = 500
    attempt_count = 0
    locked = false
    
    function authenticate(pwd):
        if pwd != password:
            attempt_count = attempt_count + 1
            if attempt_count >= 3:
                locked = true
                return {authenticated: false, message: "Account locked"}
            return {authenticated: false, message: "Wrong password"}
        
        attempt_count = 0  // Reset on successful auth
        return {authenticated: true, message: "Authenticated"}
    
    function withdraw(amount, pwd):
        auth = authenticate(pwd)
        if not auth.authenticated:
            return {success: false, message: auth.message}
        
        if locked:
            return {success: false, message: "Account locked"}
        
        if amount > transaction_limit:
            return {success: false, message: "Exceeds transaction limit"}
        
        if amount > balance:
            return {success: false, message: "Insufficient funds"}
        
        balance = balance - amount
        return {success: true, new_balance: balance}
    
    return {
        withdraw: withdraw,
        deposit: (amount, pwd) => {
            auth = authenticate(pwd)
            if not auth.authenticated:
                return {success: false, message: auth.message}
            balance = balance + amount
            return {success: true, new_balance: balance}
        }
    }

// Usage
account = make_secure_account(1000, "secret123")
account.withdraw(100, "secret123")   // {success: true}
account.withdraw(100, "wrongpwd")    // {success: false, message: "Wrong password"}
account.withdraw(100, "wrongpwd")    // 2 failures
account.withdraw(100, "wrongpwd")    // 3rd failure → {success: false, message: "Account locked"}
```

---

## Trade-Offs Table

| Factor | Functional (No State) | Imperative (With State) |
|--------|----------------------|--------------------------|
| **Code length** | Longer (state threading) | Shorter (implicit state) |
| **Readability** | Verbose for stateful domains | Natural for state changes |
| **Testing** | Simple (pure functions) | Complex (setup/teardown) |
| **Reasoning** | Substitution model | Environment model |
| **Memory** | Higher (copying state) | Lower (in-place mutation) |
| **Concurrency** | Safe without locks | Requires synchronization |
| **History** | Full (immutable) | Lost (overwritten) |
| **Composability** | Excellent (pure) | Limited (hidden state) |
| **Referential transparency** | Yes | No |
| **Debugging** | Deterministic, replay-able | Non-deterministic, hard to replay |
| **Scalability** | Better for distributed systems | Better for single-process |
| **Learning curve** | Steeper (unfamiliar to imperative programmers) | Gentler (familiar) |

---

## Summary Table

| Concept | Definition | Key Property |
|---------|-----------|--------------|
| **Assignment** | Changing a variable's binding to a new value | Enables state mutation |
| **Local state** | Mutable storage encapsulated in a closure | Private to the object |
| **Encapsulation** | Hiding internal state behind a public interface | Modularity through hiding |
| **Identity** | A unique object distinct from others with same values | Matters with mutation |
| **Referential transparency** | Expression always evaluates to same value | Lost with assignment |
| **Side effect** | Operation that modifies external state | Fundamental to stateful code |
| **Temporal reasoning** | Understanding execution sequence | Necessary with mutation |
| **Imperative** | Describing how to compute (sequence of steps) | Assignment-based style |
| **Functional** | Describing what to compute (transformations) | Avoids assignment |
| **Closure** | Function that remembers its definition environment | Enables encapsulated state |
| **Sameness** | Whether two objects are "the same" | Becomes ambiguous with mutation |
| **Mutation** | Changing a value in place | What assignment does |

---

## Key Principles

### The State Encapsulation Principle
> **Use assignment to encapsulate state within objects, exposing only a controlled interface.**

Good: Hidden internal state
```pseudocode
account = make_account(100)
account.withdraw(10)  // Internal state hidden
// account.balance would fail (not exposed)
```

Bad: Global or exposed state
```pseudocode
global_balance = 100
global_balance = global_balance - 10  // No encapsulation
```

### The Locality Principle
> **Assignment should be used for state local to a specific object, not for global state.**

Good: Local state
```pseudocode
account1 = make_account(100)
account2 = make_account(200)  // Independent local states
```

Bad: Global state
```pseudocode
global_balance = 100  // Shared by all accounts
```

### The Limited Scope Principle
> **Minimize the scope of mutable state to reduce reasoning complexity.**

Good: Tightly scoped
```pseudocode
function process_item(item):
    cache = {}  // Local, used only in this function
    // ...
```

Bad: Widely scoped
```pseudocode
global_cache = {}  // Available everywhere, hard to reason about
```

### The Explicit Interface Principle
> **Make state changes explicit through method calls, not hidden mutations.**

Good: Explicit
```pseudocode
account.withdraw(100)  // Clear that state changes
```

Bad: Hidden
```pseudocode
process(account)  // Does it change account? Unknown without reading code
```

---

## Practical Consequences

### Consequence 1: The Need for Identity

```pseudocode
// With mutable state, we need to distinguish objects
account1 = make_account(100)
account2 = make_account(100)

// Are they the same?
// Structural equality: account1.balance == account2.balance ✓
// Identity: account1 is account2 ✗

// Identity matters
account1.withdraw(10)
account2.withdraw(5)

// They're different now
account1.get_balance()  // 90
account2.get_balance()  // 95
```

### Consequence 2: The Lost History Problem

```pseudocode
// Functional: Full history preserved
states = [
    {balance: 100},
    {balance: 90},
    {balance: 70}
]
// Can access any previous state

// Imperative: History lost
account = make_account(100)
account.withdraw(10)   // history lost
account.withdraw(20)   // history lost
account.get_balance()  // 70 only, can't see 100 or 90
```

### Consequence 3: The Encapsulation vs. Transparency Trade-Off

```pseudocode
// Imperative: Encapsulated but opaque
account = make_account(100)
account.withdraw(10)
// Don't know how balance is computed or stored

// Functional: Transparent but verbose
{balance: 100} |> withdraw(10) |> deposit(5) |> get_balance
// Can see exact transformations but verbose
```

---

*Based on concepts from "Structure and Interpretation of Computer Programs" by Abelson and Sussman.*
