# Testing

## Definition

> "Testing is not about finding bugs. It's about clarifying your thinking and driving design. Tests are the first users of your code."
> — The Pragmatic Programmer

Testing is a fundamental practice that validates code behavior, drives better design, and serves as executable documentation. Pragmatic programmers view tests as an integral part of development, not an afterthought.

## Test Ruthlessly

**Test Early, Test Often, Test Automatically**

- Write tests as you write code, not after
- Run tests frequently during development
- Automate test execution to eliminate manual effort
- Treat failing tests as critical issues that must be fixed immediately
- Never release code that fails its tests

**TIP: A test that runs automatically is worth ten thousand test plans**

## Types of Tests

### Unit Testing
Tests individual components in isolation.

```pseudocode
// Test a single function's behavior
function test_calculateDiscount():
    product = createProduct(price: 100, category: "electronics")
    discount = calculateDiscount(product)

    assert discount equals 10
    assert product.price equals 100  // No side effects
```

**Purpose:**
- Verify individual functions/methods work correctly
- Catch bugs at the smallest scope
- Enable safe refactoring
- Document expected behavior

### Integration Testing
Tests how components work together.

```pseudocode
function test_orderProcessingPipeline():
    order = createOrder(items: [item1, item2])

    // Test full flow
    validationResult = validateOrder(order)
    paymentResult = processPayment(order)
    fulfillmentResult = createShipment(order)

    assert validationResult.isValid
    assert paymentResult.status equals "charged"
    assert fulfillmentResult.trackingNumber exists
```

**Purpose:**
- Verify components interact correctly
- Test data flow between modules
- Validate system-level behavior
- Catch interface mismatches

### Validation Testing
Tests user requirements and expectations.

```pseudocode
function test_userCanCompleteCheckout():
    // Simulate user journey
    user = loginUser("customer@example.com")
    addItemToCart(user, product: "laptop")
    proceedToCheckout(user)
    enterShippingInfo(user, address: validAddress)
    enterPaymentInfo(user, card: validCard)
    result = confirmOrder(user)

    assert result.success
    assert result.confirmationEmail sent
    assert user.orderHistory contains result.orderId
```

**Purpose:**
- Verify user stories and acceptance criteria
- Test from user perspective
- Validate business requirements
- Ensure system meets expectations

### Performance Testing
Tests system behavior under load.

```pseudocode
function test_apiHandlesExpectedLoad():
    requests = generateRequests(count: 1000, duration: "1 minute")

    startTime = now()
    responses = sendConcurrentRequests(requests)
    endTime = now()

    assert all responses have status 200
    assert average(responses.time) < 200 milliseconds
    assert max(responses.time) < 1 second
    assert (endTime - startTime) < 65 seconds
```

**Purpose:**
- Verify response times meet requirements
- Test system under expected load
- Identify bottlenecks and limits
- Validate scalability assumptions

## Testing Against Contract

**Design by Contract in Tests**

Every module has a contract: preconditions, postconditions, and invariants. Tests should verify these explicitly.

```pseudocode
function test_stackContract():
    stack = createStack(capacity: 3)

    // Test precondition: push requires non-full stack
    stack.push(1)
    stack.push(2)
    stack.push(3)
    assertThrows(() => stack.push(4), "StackOverflowError")

    // Test postcondition: pop returns last pushed item
    assert stack.pop() equals 3
    assert stack.pop() equals 2

    // Test invariant: size always reflects actual count
    assert stack.size() equals 1
    stack.push(5)
    assert stack.size() equals 2

    // Test precondition: pop requires non-empty stack
    stack.pop()
    stack.pop()
    assertThrows(() => stack.pop(), "StackUnderflowError")
```

**Contract Testing Benefits:**
- Makes assumptions explicit
- Documents expected behavior
- Catches violations early
- Enables stronger refactoring

## Writing Testable Code

**Design for Testability**

Code that's easy to test is usually better designed.

### Avoid Tight Coupling
```pseudocode
// Hard to test - directly creates dependency
class OrderProcessor:
    function processOrder(order):
        database = new PostgresDatabase()  // Tight coupling
        result = database.save(order)
        return result

// Easy to test - dependency injection
class OrderProcessor:
    database: Database

    function constructor(db: Database):
        this.database = db

    function processOrder(order):
        result = this.database.save(order)
        return result

// Test with mock
function test_orderProcessor():
    mockDb = createMockDatabase()
    processor = new OrderProcessor(mockDb)

    order = createTestOrder()
    processor.processOrder(order)

    assert mockDb.receivedCall("save", order)
```

### Use Pure Functions
```pseudocode
// Hard to test - depends on external state
globalCounter = 0

function incrementAndGet():
    globalCounter = globalCounter + 1
    return globalCounter

// Easy to test - pure function
function increment(value):
    return value + 1

// Test is simple and reliable
function test_increment():
    assert increment(5) equals 6
    assert increment(0) equals 1
    assert increment(-1) equals 0
```

### Separate I/O from Logic
```pseudocode
// Hard to test - logic mixed with I/O
function processUserFile(filename):
    data = readFile(filename)  // I/O
    total = 0
    for line in data.lines:
        number = parseNumber(line)
        if number > 0:
            total = total + number
    writeFile("output.txt", total)  // I/O
    return total

// Easy to test - separated concerns
function calculatePositiveSum(numbers):
    total = 0
    for number in numbers:
        if number > 0:
            total = total + number
    return total

function processUserFile(filename):
    data = readFile(filename)
    numbers = parseNumbers(data)
    total = calculatePositiveSum(numbers)
    writeFile("output.txt", total)
    return total

// Test logic without I/O
function test_calculatePositiveSum():
    assert calculatePositiveSum([1, 2, 3]) equals 6
    assert calculatePositiveSum([1, -2, 3]) equals 4
    assert calculatePositiveSum([-1, -2, -3]) equals 0
    assert calculatePositiveSum([]) equals 0
```

## Test Coverage - What to Measure

**Coverage Metrics Should Guide, Not Rule**

- **Line Coverage**: Percentage of code lines executed by tests
- **Branch Coverage**: Percentage of decision branches tested
- **Path Coverage**: Percentage of execution paths tested
- **Mutation Coverage**: Percentage of introduced bugs caught

```pseudocode
function calculateGrade(score):
    if score >= 90:
        return "A"
    else if score >= 80:
        return "B"
    else if score >= 70:
        return "C"
    else:
        return "F"

// 50% branch coverage (only tests two branches)
function test_calculateGrade_partial():
    assert calculateGrade(95) equals "A"
    assert calculateGrade(60) equals "F"

// 100% branch coverage (tests all branches)
function test_calculateGrade_complete():
    assert calculateGrade(95) equals "A"
    assert calculateGrade(85) equals "B"
    assert calculateGrade(75) equals "C"
    assert calculateGrade(60) equals "F"

    // Boundary testing
    assert calculateGrade(90) equals "A"
    assert calculateGrade(89) equals "B"
    assert calculateGrade(80) equals "B"
    assert calculateGrade(79) equals "C"
```

**TIP: High coverage doesn't guarantee good tests, but low coverage guarantees inadequate tests**

Focus on:
- Critical business logic
- Error handling paths
- Boundary conditions
- State transitions
- Integration points

## Tests as Documentation

**Tests Describe How Code Should Be Used**

Well-written tests serve as executable examples and living documentation.

```pseudocode
// Test documents API usage patterns
function test_shoppingCart_documentation():
    // Creating and using a shopping cart
    cart = createCart()
    assert cart.isEmpty()

    // Adding items
    laptop = createProduct(id: "L1", price: 999.99)
    mouse = createProduct(id: "M1", price: 29.99)

    cart.addItem(laptop, quantity: 1)
    cart.addItem(mouse, quantity: 2)

    // Checking contents
    assert cart.itemCount() equals 3
    assert cart.total() equals 1059.97

    // Applying discounts
    cart.applyPromoCode("SAVE10")
    assert cart.total() equals 953.97

    // Removing items
    cart.removeItem(mouse)
    assert cart.itemCount() equals 1

    // Clearing cart
    cart.clear()
    assert cart.isEmpty()
```

**Tests Answer Questions:**
- How do I create an instance?
- What parameters does this function accept?
- What does this function return?
- How do components interact?
- What happens in error cases?

## Test Organization Best Practices

### Arrange-Act-Assert Pattern
```pseudocode
function test_transferFunds():
    // Arrange - set up test conditions
    sourceAccount = createAccount(balance: 1000)
    targetAccount = createAccount(balance: 500)

    // Act - perform the operation
    result = transferFunds(
        from: sourceAccount,
        to: targetAccount,
        amount: 200
    )

    // Assert - verify results
    assert result.success
    assert sourceAccount.balance equals 800
    assert targetAccount.balance equals 700
```

### One Logical Assertion Per Test
```pseudocode
// Poor - multiple unrelated assertions
function test_user_everything():
    user = createUser("john@example.com")
    assert user.email equals "john@example.com"
    assert user.validatePassword("pass123")
    assert user.cart.isEmpty()
    assert user.orderHistory.count equals 0

// Better - focused tests
function test_user_createdWithCorrectEmail():
    user = createUser("john@example.com")
    assert user.email equals "john@example.com"

function test_user_canValidatePassword():
    user = createUser("john@example.com")
    user.setPassword("pass123")
    assert user.validatePassword("pass123")

function test_user_startsWithEmptyCart():
    user = createUser("john@example.com")
    assert user.cart.isEmpty()

function test_user_startsWithNoOrderHistory():
    user = createUser("john@example.com")
    assert user.orderHistory.count equals 0
```

### Test Error Conditions
```pseudocode
function test_divideByZero_throwsError():
    assertThrows(() => divide(10, 0), "DivisionByZeroError")

function test_invalidEmail_rejectsRegistration():
    result = registerUser("not-an-email")
    assert result.success equals false
    assert result.error equals "Invalid email format"

function test_insufficientFunds_preventsWithdrawal():
    account = createAccount(balance: 100)
    result = account.withdraw(150)

    assert result.success equals false
    assert account.balance equals 100  // Unchanged
    assert result.error contains "Insufficient funds"
```

## Summary Table

| Testing Principle | Purpose | Implementation |
|------------------|---------|----------------|
| **Test Early** | Catch bugs when cheapest to fix | Write tests during development |
| **Test Often** | Maintain confidence in changes | Run tests automatically and frequently |
| **Unit Tests** | Verify individual components | Test functions/methods in isolation |
| **Integration Tests** | Verify component interactions | Test data flow between modules |
| **Validation Tests** | Verify user requirements | Test user stories and acceptance criteria |
| **Performance Tests** | Verify non-functional requirements | Test response times and throughput |
| **Contract Testing** | Verify preconditions/postconditions | Test assumptions explicitly |
| **Testable Design** | Make testing easier | Use dependency injection, pure functions |
| **Coverage Metrics** | Guide test effort | Measure but don't obsess over percentages |
| **Tests as Docs** | Document API usage | Write clear, example-driven tests |
| **AAA Pattern** | Organize tests clearly | Arrange, Act, Assert structure |
| **Focused Tests** | Enable precise debugging | One logical assertion per test |
| **Error Testing** | Verify failure handling | Test exception paths and edge cases |

---

*Based on concepts from "The Pragmatic Programmer" by David Thomas and Andrew Hunt.*
