# The Boy Scout Rule

## Definition

> "Leave the campground cleaner than you found it."
> — Boy Scouts of America (applied to code by Robert C. Martin)

Every time you touch code, make it a little better. Small, incremental improvements compound over time.

## The Principle

When you check in code, it should be cleaner than when you checked it out. Not necessarily perfect—just better.

This applies even when:
- You're only fixing a small bug
- You're just adding a minor feature
- You didn't write the original code
- You're under time pressure

## Why It Matters

### Without the Boy Scout Rule

```
Day 1:   Code quality: ████████░░ (80%)
Day 30:  Code quality: ███████░░░ (70%)
Day 90:  Code quality: █████░░░░░ (50%)
Day 180: Code quality: ███░░░░░░░ (30%)

→ Codebase becomes unmaintainable
→ Developers avoid touching code
→ Technical debt compounds
```

### With the Boy Scout Rule

```
Day 1:   Code quality: ████████░░ (80%)
Day 30:  Code quality: ████████░░ (82%)
Day 90:  Code quality: █████████░ (88%)
Day 180: Code quality: █████████░ (92%)

→ Codebase improves over time
→ Developers enjoy working on code
→ Technical debt decreases
```

## What to Improve

Small improvements that don't change behavior:

### Rename Something

```pseudocode
// Before
function calc(a, b) {
    return a * b * 0.15
}

// After - clearer name
function calculateTax(subtotal, taxRate) {
    return subtotal * taxRate
}
```

### Extract a Method

```pseudocode
// Before
function processOrder(order) {
    // ... lots of code ...

    // Calculate total
    subtotal = 0
    for item in order.items {
        subtotal += item.price * item.quantity
    }
    tax = subtotal * TAX_RATE
    total = subtotal + tax

    // ... more code ...
}

// After - extracted
function processOrder(order) {
    // ... lots of code ...

    total = calculateOrderTotal(order)

    // ... more code ...
}

function calculateOrderTotal(order) {
    subtotal = sumItemPrices(order.items)
    tax = subtotal * TAX_RATE
    return subtotal + tax
}
```

### Add a Clarifying Comment (If Necessary)

```pseudocode
// Before
regex = /^(?=.*[A-Z])(?=.*[0-9]).{8,}$/

// After
// Password must be 8+ chars with at least one uppercase and one digit
PASSWORD_PATTERN = /^(?=.*[A-Z])(?=.*[0-9]).{8,}$/
```

### Remove Dead Code

```pseudocode
// Before
function processUser(user) {
    // oldProcess(user)  // Deprecated in v2.0
    // if (useNewProcess) {
        newProcess(user)
    // }
}

// After
function processUser(user) {
    newProcess(user)
}
```

### Fix a Small Bug Nearby

```pseudocode
// While fixing something else, you notice:
// Before
if (count = 0) {  // Bug: assignment instead of comparison
    handleEmpty()
}

// After
if (count == 0) {
    handleEmpty()
}
```

### Add a Missing Test

```pseudocode
// You notice an edge case isn't tested
function testDivide_byZero_throwsException() {
    assertThrows(() => divide(10, 0))
}
```

### Reduce Duplication

```pseudocode
// Before
function sendWelcomeEmail(user) {
    subject = "Welcome!"
    body = "Hello " + user.name + "..."
    send(user.email, subject, body)
}

function sendResetEmail(user) {
    subject = "Password Reset"
    body = "Hello " + user.name + "..."
    send(user.email, subject, body)
}

// After - extract common pattern
function sendWelcomeEmail(user) {
    sendEmail(user, "Welcome!", buildWelcomeBody(user))
}

function sendResetEmail(user) {
    sendEmail(user, "Password Reset", buildResetBody(user))
}

function sendEmail(user, subject, body) {
    send(user.email, subject, body)
}
```

## What NOT to Do

### Don't Make Large Changes Unrelated to Your Task

```pseudocode
// You're fixing a small bug in UserService
// DON'T also:
// - Rewrite the entire authentication system
// - Change the database schema
// - Refactor 20 other files
```

### Don't Break Things

The rule is to leave code **cleaner**, not different-but-broken.

```pseudocode
// DON'T change working behavior
// Before (works)
function isAdult(age) {
    return age >= 21  // Company policy is 21
}

// Don't "fix" to 18 without understanding why it's 21
```

### Don't Gold-Plate

Small improvements, not perfection.

```pseudocode
// You're fixing a typo in an error message
// DON'T spend 4 hours abstracting the entire error handling system
```

## Practical Guidelines

### Time Boxing

- Spend 5-15 minutes on improvements
- If it takes longer, create a separate task/ticket
- Don't let "cleaning" delay your main work

### Scope Limiting

- Only improve code you're already touching
- Stay within the same file or closely related files
- Don't go hunting for problems elsewhere

### Safety

- Run tests after every change
- Commit improvements separately from feature work
- If unsure, don't change it

## Team Adoption

### Commit Strategy

```bash
# Separate cleanup commits from feature commits
git commit -m "refactor: rename calculate() to calculateTax()"
git commit -m "feat: add support for multiple tax rates"
```

### Code Review

- Praise cleanup efforts
- Don't demand perfection
- Accept incremental improvement

### Definition of Done

Include "code cleaner than before" in your Definition of Done:

- [ ] Feature implemented
- [ ] Tests passing
- [ ] **Code quality improved (or maintained)**
- [ ] Documentation updated

## The Compound Effect

Small improvements compound dramatically:

| Improvement/Day | 1 Year Total |
|-----------------|--------------|
| 1 rename | 250 better names |
| 1 extracted method | 250 cleaner functions |
| 1 deleted dead line | 250 lines removed |
| 1 added test | 250 new tests |

## Quotes

> "The only way to go fast is to go well."
> — Robert C. Martin

> "Always leave the code better than you found it."
> — Robert C. Martin

> "Any fool can write code that a computer can understand. Good programmers write code that humans can understand."
> — Martin Fowler

## Summary

1. **Touch it? Improve it.** Every file you open.
2. **Small changes.** 5-15 minutes max.
3. **Don't break things.** Tests must pass.
4. **Separate commits.** Cleanup vs. features.
5. **Compound over time.** Small + consistent = huge.

---

*Based on concepts from "Clean Code" by Robert C. Martin.*
