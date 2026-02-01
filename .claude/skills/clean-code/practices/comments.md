# Comments

## Core Principle

> "The proper use of comments is to compensate for our failure to express ourselves in code."
> — Robert C. Martin

Comments are, at best, a necessary evil. The best comment is the one you found a way not to write.

## Why Comments Fail

- **Comments lie**: Code changes, comments don't always follow
- **Comments decay**: They become stale as code evolves
- **Comments excuse**: They compensate for bad code instead of fixing it

## Good Comments

Some comments are necessary and beneficial.

### Legal Comments

```pseudocode
// Copyright (c) 2024 Company Name. All rights reserved.
// Licensed under the MIT License. See LICENSE file.
```

### Informative Comments

Explain something that can't be expressed in code:

```pseudocode
// Format: YYYY-MM-DD HH:MM:SS
pattern = "\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}"

// Matches US phone numbers: (xxx) xxx-xxxx
phoneRegex = /\(\d{3}\) \d{3}-\d{4}/
```

### Explanation of Intent

Why, not what:

```pseudocode
// We use insertion sort here because the list is almost always
// nearly sorted, making it O(n) in practice vs O(n log n) for quicksort
function sortRecentlyModified(items) {
    return insertionSort(items)
}
```

### Clarification

When you can't change the code (third-party library):

```pseudocode
// The API returns null instead of empty array when no results
results = api.search(query)
items = results ?? []  // Handle API quirk
```

### Warning of Consequences

```pseudocode
// WARNING: This test takes 30 minutes to run.
// Only run it in the nightly build.
function testPerformanceUnderLoad() { ... }

// Don't run this in production - it will drop all tables
function resetDatabase() { ... }
```

### TODO Comments

Mark work that needs to be done (but track and address them):

```pseudocode
// TODO: Remove this after the v2 API migration (ticket #1234)
function legacyApiCall() { ... }

// FIXME: This breaks for dates before 1970
function parseDate(dateString) { ... }
```

### Documentation Comments (Public APIs)

```pseudocode
/**
 * Calculates compound interest.
 *
 * @param principal - Initial investment amount
 * @param rate - Annual interest rate (e.g., 0.05 for 5%)
 * @param years - Number of years
 * @param compoundingPerYear - How often interest compounds (default: 12)
 * @returns Total amount after interest
 *
 * @example
 * calculateCompoundInterest(1000, 0.05, 10) // Returns 1647.01
 */
function calculateCompoundInterest(principal, rate, years, compoundingPerYear = 12) {
    return principal * (1 + rate / compoundingPerYear) ^ (compoundingPerYear * years)
}
```

## Bad Comments

Most comments fall into these categories.

### Mumbling

```pseudocode
// Bad - what does this mean?
try {
    loadProperties()
} catch (Exception e) {
    // No properties means default values
}
// What default values? Where? What happens?
```

### Redundant Comments

```pseudocode
// Bad - says exactly what the code says
i = i + 1  // Increment i

// Bad - restates the obvious
// This function returns the name
function getName() {
    return name
}

// Bad - JavaDoc that adds nothing
/**
 * The name
 * @return The name
 */
function getName() {
    return name
}
```

### Misleading Comments

```pseudocode
// Bad - comment lies
// Returns true if the user is active
function checkUser(user) {
    return user != null && user.status == "ACTIVE" && !user.isDeleted
    // Actually checks three things, not just "active"
}
```

### Mandated Comments

Don't require comments on every function:

```pseudocode
// Bad - mandated but useless
/**
 * @param name The name
 * @param age The age
 * @return The user
 */
function createUser(name, age) {
    return new User(name, age)
}
```

### Journal Comments

```pseudocode
// Bad - use version control instead
// 2024-01-15: Added validation
// 2024-01-20: Fixed bug in validation
// 2024-02-01: Refactored to use regex
function validate(input) { ... }
```

### Noise Comments

```pseudocode
// Bad - just noise
// Default constructor
constructor() { }

// The day of the month
dayOfMonth = 0

// Returns the day of the month
function getDayOfMonth() {
    return dayOfMonth
}
```

### Position Markers

```pseudocode
// Bad - don't use comment banners
// /////////////////// GETTERS /////////////////////

function getName() { return name }
function getAge() { return age }

// /////////////////// SETTERS /////////////////////

function setName(n) { name = n }
function setAge(a) { age = a }
```

### Closing Brace Comments

```pseudocode
// Bad - indicates function is too long
function process() {
    if (condition) {
        for (item in items) {
            // lots of code...
        } // for
    } // if
} // process

// Good - make function shorter instead
```

### Attributed/Byline Comments

```pseudocode
// Bad - use version control
// Added by John on 2024-01-15
function newFeature() { ... }
```

### Commented-Out Code

```pseudocode
// Bad - delete it, version control has history
function calculate() {
    // oldValue = legacyCalculation()
    // if (oldValue < 0) {
    //     oldValue = 0
    // }
    return newCalculation()
}
```

### HTML/Formatting in Comments

```pseudocode
// Bad - hard to read in code
/**
 * <p>This method does <b>important</b> things:</p>
 * <ul>
 *   <li>First thing</li>
 *   <li>Second thing</li>
 * </ul>
 */
```

### Too Much Information

```pseudocode
// Bad - encyclopedia entry
/*
 * RFC 2045 - Multipurpose Internet Mail Extensions (MIME)
 * Part One: Format of Internet Message Bodies
 * Section 6.8: Base64 Content-Transfer-Encoding
 *
 * The encoding process represents 24-bit groups of input bits
 * as output strings of 4 encoded characters...
 * [200 more lines]
 */
function encodeBase64(data) { ... }
```

## Replace Comments with Better Code

### Extract Method

```pseudocode
// Bad
// Check if employee is eligible for benefits
if (employee.type == "FULL_TIME" && employee.tenure > 90 && !employee.isContractor) {
    // ...
}

// Good
if (employee.isEligibleForBenefits()) {
    // ...
}
```

### Use Better Names

```pseudocode
// Bad
int d; // elapsed time in days

// Good
int elapsedTimeInDays;
```

### Use Constants

```pseudocode
// Bad
if (age >= 21) {  // Legal drinking age in US

// Good
LEGAL_DRINKING_AGE = 21
if (age >= LEGAL_DRINKING_AGE) {
```

### Create Explaining Variables

```pseudocode
// Bad
// Check for valid email format
if (input.match(/^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/)) {

// Good
isValidEmail = emailValidator.validate(input)
if (isValidEmail) {
```

## Summary

### Write a Comment When:
- Legal requirements demand it
- Explaining WHY (not what)
- Warning of non-obvious consequences
- Clarifying third-party code you can't change
- API documentation for public interfaces

### Don't Write a Comment When:
- Code can express the same thing
- It restates what code does
- It will become stale
- It's a crutch for bad naming
- It's commented-out code
- It's a journal/changelog

---

*Based on concepts from "Clean Code" by Robert C. Martin.*
