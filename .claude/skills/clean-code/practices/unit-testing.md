# Unit Testing

## Core Principle

> "Test code is just as important as production code."
> — Robert C. Martin

Tests enable change. Without tests, every change is a potential bug. With tests, you can refactor with confidence.

## The Three Laws of TDD

1. **You may not write production code until you have written a failing unit test.**
2. **You may not write more of a unit test than is sufficient to fail.**
3. **You may not write more production code than is sufficient to pass the current test.**

This creates a tight feedback loop: test → code → refactor → repeat.

## F.I.R.S.T. Principles

Clean tests follow five rules:

### Fast

Tests should run quickly. Slow tests don't get run.

```pseudocode
// Bad - slow test
function testUserCreation() {
    // Connects to real database
    user = database.createUser(data)
    assert(database.find(user.id) != null)
}

// Good - fast test with mock
function testUserCreation() {
    mockDatabase = new MockDatabase()
    service = new UserService(mockDatabase)

    service.createUser(data)

    assert(mockDatabase.saveCalled)
    assert(mockDatabase.savedUser.name == data.name)
}
```

### Independent

Tests should not depend on each other. Run in any order.

```pseudocode
// Bad - tests depend on each other
function test1_createUser() {
    userId = service.createUser(data)
    // test2 depends on this userId
}

function test2_updateUser() {
    service.updateUser(userId, newData)  // Depends on test1!
}

// Good - independent tests
function testCreateUser() {
    userId = service.createUser(data)
    assert(userId != null)
}

function testUpdateUser() {
    userId = service.createUser(data)  // Own setup
    service.updateUser(userId, newData)
    assert(service.getUser(userId).name == newData.name)
}
```

### Repeatable

Tests should produce the same result every time, in any environment.

```pseudocode
// Bad - depends on current time
function testIsExpired() {
    subscription = new Subscription(expiresAt: "2024-12-31")
    assert(subscription.isExpired() == false)  // Fails after 2024!
}

// Good - inject time dependency
function testIsExpired() {
    clock = new FakeClock("2024-06-15")
    subscription = new Subscription(expiresAt: "2024-12-31", clock)

    assert(subscription.isExpired() == false)
}

function testIsExpiredWhenPastDate() {
    clock = new FakeClock("2025-01-15")
    subscription = new Subscription(expiresAt: "2024-12-31", clock)

    assert(subscription.isExpired() == true)
}
```

### Self-Validating

Tests should have a boolean output: pass or fail. No manual interpretation.

```pseudocode
// Bad - requires manual inspection
function testCalculation() {
    result = calculate(input)
    print(result)  // Developer must check if this looks right
}

// Good - clear pass/fail
function testCalculation() {
    result = calculate(input)
    assert(result == expectedValue)
}
```

### Timely

Write tests at the right time—before the production code (TDD).

## One Assert Per Test

Each test should verify one concept. This makes failures clear.

```pseudocode
// Bad - multiple asserts, unclear what failed
function testUser() {
    user = createUser(data)

    assert(user.id != null)
    assert(user.name == "John")
    assert(user.email == "john@example.com")
    assert(user.createdAt != null)
    assert(user.isActive == true)
}

// Good - one concept per test
function testUserHasId() {
    user = createUser(data)
    assert(user.id != null)
}

function testUserHasCorrectName() {
    user = createUser(data)
    assert(user.name == "John")
}

function testUserIsActiveByDefault() {
    user = createUser(data)
    assert(user.isActive == true)
}
```

Exception: When testing a single concept requires multiple assertions:

```pseudocode
// OK - testing one concept (coordinates)
function testMoveUpdatesPosition() {
    player = new Player(x: 0, y: 0)

    player.move(Direction.NORTH)

    assert(player.x == 0)
    assert(player.y == 1)  // Both needed to verify position
}
```

## Single Concept Per Test

Test one logical concept, even if it requires setup:

```pseudocode
// Bad - testing multiple concepts
function testShoppingCart() {
    cart = new Cart()

    // Concept 1: Adding items
    cart.addItem(item1)
    assert(cart.itemCount == 1)

    // Concept 2: Calculating total
    cart.addItem(item2)
    assert(cart.total == item1.price + item2.price)

    // Concept 3: Applying discount
    cart.applyDiscount(10)
    assert(cart.total == (item1.price + item2.price) * 0.9)
}

// Good - separate tests
function testAddItemIncreasesCount() {
    cart = new Cart()
    cart.addItem(item)
    assert(cart.itemCount == 1)
}

function testTotalSumsItemPrices() {
    cart = new Cart()
    cart.addItem(item1)
    cart.addItem(item2)
    assert(cart.total == item1.price + item2.price)
}

function testDiscountReducesTotal() {
    cart = new Cart()
    cart.addItem(item1)
    originalTotal = cart.total

    cart.applyDiscount(10)

    assert(cart.total == originalTotal * 0.9)
}
```

## Test Structure: Arrange-Act-Assert

```pseudocode
function testWithdrawReducesBalance() {
    // Arrange - set up test data
    account = new Account(balance: 100)

    // Act - perform the action
    account.withdraw(30)

    // Assert - verify the result
    assert(account.balance == 70)
}
```

Also known as:
- **Given-When-Then** (BDD)
- **Setup-Exercise-Verify** (xUnit)

## Test Naming

Names should describe what is being tested:

```pseudocode
// Pattern: methodName_condition_expectedBehavior

function withdraw_sufficientFunds_reducesBalance()
function withdraw_insufficientFunds_throwsException()
function withdraw_negativeAmount_throwsException()
function withdraw_zero_noChange()

// Or descriptive sentences
function test_user_can_withdraw_when_balance_is_sufficient()
function test_withdrawal_fails_when_balance_is_insufficient()
```

## Test-Driven Development (TDD) Cycle

```
1. RED:    Write a failing test
2. GREEN:  Write minimal code to pass
3. REFACTOR: Clean up while keeping tests green
```

### Example TDD Session

```pseudocode
// Step 1: RED - Write failing test
function testStackIsEmptyWhenCreated() {
    stack = new Stack()
    assert(stack.isEmpty() == true)
}
// Fails: Stack class doesn't exist

// Step 2: GREEN - Minimal code to pass
class Stack {
    function isEmpty() {
        return true
    }
}
// Test passes!

// Step 3: RED - Next test
function testStackIsNotEmptyAfterPush() {
    stack = new Stack()
    stack.push("item")
    assert(stack.isEmpty() == false)
}
// Fails: push doesn't exist, isEmpty always returns true

// Step 4: GREEN - Minimal code
class Stack {
    size = 0

    function isEmpty() {
        return size == 0
    }

    function push(item) {
        size++
    }
}
// Test passes!

// Continue cycle...
```

## Test Doubles

### Types

| Type | Purpose |
|------|---------|
| **Dummy** | Passed but never used |
| **Stub** | Provides canned answers |
| **Spy** | Records calls for verification |
| **Mock** | Verifies expected interactions |
| **Fake** | Working implementation (in-memory DB) |

### Example

```pseudocode
// Stub - returns canned data
class StubUserRepository {
    function findById(id) {
        return new User(id: id, name: "Test User")
    }
}

// Mock - verifies interactions
class MockEmailService {
    sendCalled = false
    lastRecipient = null

    function send(to, subject, body) {
        sendCalled = true
        lastRecipient = to
    }

    function verifySentTo(email) {
        assert(sendCalled)
        assert(lastRecipient == email)
    }
}

// Using test doubles
function testRegistrationSendsWelcomeEmail() {
    userRepo = new StubUserRepository()
    emailService = new MockEmailService()
    service = new RegistrationService(userRepo, emailService)

    service.register("test@example.com", "password")

    emailService.verifySentTo("test@example.com")
}
```

## Clean Test Code

Tests should be as clean as production code:

```pseudocode
// Bad - messy test
function testX() {
    a = new A(); b = new B(); c = new C()
    a.setB(b); b.setC(c); c.setValue(42)
    r = a.doThing()
    assert(r == 42)
}

// Good - readable test
function testCalculationReturnsCValue() {
    calculator = createCalculatorWithValue(42)

    result = calculator.calculate()

    assert(result == 42)
}

function createCalculatorWithValue(value) {
    innerComponent = new InnerComponent(value)
    middleComponent = new MiddleComponent(innerComponent)
    return new Calculator(middleComponent)
}
```

## Test Coverage

- Aim for high coverage, but don't worship 100%
- Focus on critical paths and edge cases
- Missing tests > bad tests > no tests
- Use coverage tools to find gaps, not as a metric

## Summary Checklist

- [ ] Tests are fast (< 1 second each)
- [ ] Tests are independent (no shared state)
- [ ] Tests are repeatable (deterministic)
- [ ] Tests self-validate (boolean result)
- [ ] One concept per test
- [ ] Clear Arrange-Act-Assert structure
- [ ] Descriptive test names
- [ ] Test code is clean and readable

---

*Based on concepts from "Clean Code" by Robert C. Martin.*
