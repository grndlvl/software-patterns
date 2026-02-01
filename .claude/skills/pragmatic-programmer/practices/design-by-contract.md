# Design by Contract (DbC)

## Definition

> "Nothing astonishes men so much as common sense and plain dealing." - Ralph Waldo Emerson

**Design by Contract** is a software design approach where software components collaborate based on clearly defined agreements of rights and responsibilities. Each party in a software contract has obligations and benefits, creating a formal agreement about what a routine expects and what it guarantees in return.

The contract defines:
- **What the routine expects** (preconditions)
- **What the routine guarantees** (postconditions)
- **What the routine maintains** (invariants)

## Preconditions

**Preconditions** are requirements that must be true before a routine can be called. They define the caller's obligations.

### Characteristics
- Define valid inputs and system state
- Caller's responsibility to satisfy
- Routine has no obligation to handle violations
- Should fail fast if violated

### Examples
```pseudocode
function sqrt(value)
  precondition: value >= 0
  ...
```

```pseudocode
function withdraw(account, amount)
  precondition: account.isOpen == true
  precondition: account.balance >= amount
  precondition: amount > 0
  ...
```

```pseudocode
function processOrder(order)
  precondition: order != null
  precondition: order.items.length > 0
  precondition: order.customer.isVerified == true
  ...
```

## Postconditions

**Postconditions** are guarantees about the state after a routine completes successfully. They define the routine's obligations.

### Characteristics
- Define what the routine promises to deliver
- Routine's responsibility to ensure
- Must be true for all possible valid inputs
- Define the result and side effects

### Examples
```pseudocode
function deposit(account, amount)
  precondition: amount > 0
  postcondition: account.balance == old(account.balance) + amount
  postcondition: account.lastTransaction == current_time()
```

```pseudocode
function sort(array)
  postcondition: array.isSorted() == true
  postcondition: array.length == old(array.length)
  postcondition: array.containsAllElementsFrom(old(array)) == true
```

```pseudocode
function createUser(email, password)
  precondition: email.isValid()
  precondition: password.length >= 8
  postcondition: result.id != null
  postcondition: result.email == email
  postcondition: result.passwordHash != password  // password was hashed
  postcondition: database.contains(result) == true
```

## Invariants

**Invariants** are conditions that must always be true for an object or system, both before and after any operation.

### Characteristics
- Define consistent state
- Must hold at construction
- Must hold before and after every public method
- May be temporarily violated during private method execution
- Restored before method exits

### Examples
```pseudocode
class BankAccount
  invariant: balance >= 0
  invariant: accountNumber.length == 10
  invariant: transactions.length >= 0
  invariant: sum(transactions.amounts) == balance
```

```pseudocode
class CircularBuffer
  invariant: 0 <= readPosition < capacity
  invariant: 0 <= writePosition < capacity
  invariant: 0 <= size <= capacity
  invariant: buffer.length == capacity
```

```pseudocode
class SortedList
  invariant: forall i, j where i < j: elements[i] <= elements[j]
  invariant: size == elements.length
  invariant: size >= 0
```

## Benefits of Design by Contract

| Benefit | Description |
|---------|-------------|
| **Clarity** | Explicit documentation of expectations and guarantees |
| **Correctness** | Clear definition of what "correct" means for each routine |
| **Early Detection** | Bugs caught at the boundary where they occur |
| **Better Testing** | Contracts define test cases automatically |
| **Safer Refactoring** | Contracts must be maintained, preserving behavior |
| **Reduced Defensive Code** | No need to check what the contract guarantees |
| **Documentation** | Self-documenting through executable specifications |
| **Reliability** | Components can trust their collaborators |

## Implementing DbC Without Language Support

Many languages don't have built-in contract support (unlike Eiffel), but you can implement DbC principles:

### 1. Assertion-Based Contracts

```pseudocode
function withdraw(account, amount)
  // Preconditions
  assert(account.isOpen, "Account must be open")
  assert(account.balance >= amount, "Insufficient funds")
  assert(amount > 0, "Amount must be positive")

  oldBalance = account.balance

  // Method body
  account.balance = account.balance - amount
  account.lastTransaction = currentTime()

  // Postconditions
  assert(account.balance == oldBalance - amount, "Balance incorrectly updated")
  assert(account.lastTransaction != null, "Transaction time not recorded")

  // Invariant check
  assert(account.balance >= 0, "Invariant violated: negative balance")
```

### 2. Guard Clauses with Explicit Errors

```pseudocode
function processPayment(order, paymentMethod)
  // Preconditions as guard clauses
  if order == null:
    throw ContractViolation("Precondition failed: order cannot be null")

  if order.total <= 0:
    throw ContractViolation("Precondition failed: order total must be positive")

  if not paymentMethod.isValid():
    throw ContractViolation("Precondition failed: invalid payment method")

  // Process payment
  transaction = paymentMethod.charge(order.total)
  order.status = "PAID"

  // Postconditions
  if transaction.status != "SUCCESS":
    throw ContractViolation("Postcondition failed: payment not successful")

  if order.status != "PAID":
    throw ContractViolation("Postcondition failed: order status not updated")

  return transaction
```

### 3. Decorator/Wrapper Pattern

```pseudocode
function withContract(preconditions, postconditions, invariants)
  return function(originalFunction)
    return function wrappedFunction(arguments)
      // Check invariants before
      for each invariant in invariants:
        assert(invariant(this), "Invariant violated before call")

      // Check preconditions
      for each precondition in preconditions:
        assert(precondition(arguments), "Precondition failed")

      // Capture old state for postconditions
      oldState = captureState(this)

      // Execute original function
      result = originalFunction.apply(this, arguments)

      // Check postconditions
      for each postcondition in postconditions:
        assert(postcondition(result, arguments, oldState), "Postcondition failed")

      // Check invariants after
      for each invariant in invariants:
        assert(invariant(this), "Invariant violated after call")

      return result
```

### 4. Documentation Conventions

```pseudocode
/**
 * Withdraws the specified amount from the account.
 *
 * @requires account.isOpen == true
 * @requires account.balance >= amount
 * @requires amount > 0
 * @ensures account.balance == old(account.balance) - amount
 * @ensures account.lastTransaction != null
 * @invariant account.balance >= 0
 */
function withdraw(account, amount)
  // Implementation
```

## Design by Contract vs. Defensive Programming

### Key Differences

| Aspect | Design by Contract | Defensive Programming |
|--------|-------------------|----------------------|
| **Philosophy** | Trust but verify at boundaries | Trust nothing |
| **Responsibility** | Caller ensures preconditions | Routine checks everything |
| **Duplicate Checks** | Minimal, at contract boundaries | Extensive, everywhere |
| **Performance** | Faster (fewer checks) | Slower (redundant checks) |
| **Failure Mode** | Fail fast at violation point | May propagate invalid state |
| **Code Clarity** | Clear responsibilities | Mixed concerns |

### When to Use Each

**Design by Contract:**
- Internal APIs and module boundaries
- When caller and callee are under your control
- Performance-critical code
- Clear ownership of components

**Defensive Programming:**
- Public APIs exposed to external users
- Input from untrusted sources
- Security-sensitive operations
- User-facing validation

### Combined Approach

```pseudocode
// Public API - Defensive
function publicWithdraw(accountId, amount)
  // Defensive: validate all inputs
  if accountId == null or not isValidAccountId(accountId):
    return Error("Invalid account ID")

  if amount <= 0:
    return Error("Amount must be positive")

  account = loadAccount(accountId)
  if account == null:
    return Error("Account not found")

  // Call internal method with contract
  return internalWithdraw(account, amount)

// Internal method - DbC
function internalWithdraw(account, amount)
  // Contract: caller guarantees these
  precondition: account != null
  precondition: account.isOpen == true
  precondition: account.balance >= amount
  precondition: amount > 0

  oldBalance = account.balance
  account.balance = account.balance - amount

  postcondition: account.balance == oldBalance - amount
  invariant: account.balance >= 0
```

## Practical Guidelines

### 1. Start with Preconditions
Define what you need before doing anything else.

### 2. Make Contracts Explicit
Document them clearly, even if not enforceable by the language.

### 3. Check Contracts in Development
Use assertions that can be disabled in production if needed.

### 4. Fail Fast
Don't try to recover from contract violations.

### 5. Keep Contracts Simple
Complex contracts are hard to maintain and understand.

### 6. Contract Inheritance Rules
- Preconditions can only be weakened in subclasses
- Postconditions can only be strengthened in subclasses
- Invariants must be maintained by subclasses

```pseudocode
class Shape
  function area()
    postcondition: result >= 0

class Circle extends Shape
  function area()
    postcondition: result >= 0           // Must maintain parent postcondition
    postcondition: result == PI * r * r  // Can add stronger guarantee
```

## Summary Table

| Concept | Who Ensures | When Checked | Violation Means |
|---------|-------------|--------------|-----------------|
| **Precondition** | Caller | Before routine executes | Caller's bug |
| **Postcondition** | Routine | After routine completes | Routine's bug |
| **Invariant** | Class | Before and after all public methods | Class design bug |
| **Guard Clause** | Routine | At entry point | Input validation |
| **Assertion** | Runtime | During execution | Logic error |

## Key Principles

1. **Clear Responsibilities**: Contracts make explicit who is responsible for what
2. **No Redundant Checks**: If caller guarantees precondition, routine doesn't recheck
3. **Fail Fast**: Contract violations indicate bugs, not recoverable errors
4. **Trust Within Boundaries**: Internal code can trust contracts; external input needs validation
5. **Executable Documentation**: Contracts are verified specifications

---

*Based on concepts from "The Pragmatic Programmer" by David Thomas and Andrew Hunt.*
