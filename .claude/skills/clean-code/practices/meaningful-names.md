# Meaningful Names

## Core Principle

> "The name of a variable, function, or class should answer all the big questions. It should tell you why it exists, what it does, and how it is used."
> — Robert C. Martin

Names are the primary way we communicate intent in code. Good names make code self-documenting.

## Rules for Good Names

### 1. Use Intention-Revealing Names

The name should tell you why it exists and what it does.

```pseudocode
// Bad
d = 0  // elapsed time in days

// Good
elapsedTimeInDays = 0
daysSinceCreation = 0
fileAgeInDays = 0
```

```pseudocode
// Bad
function getThem() {
    list1 = []
    for x in theList {
        if x[0] == 4 {
            list1.add(x)
        }
    }
    return list1
}

// Good
function getFlaggedCells() {
    flaggedCells = []
    for cell in gameBoard {
        if cell.isFlagged() {
            flaggedCells.add(cell)
        }
    }
    return flaggedCells
}
```

### 2. Avoid Disinformation

Don't use names that obscure meaning or mislead.

```pseudocode
// Bad - not actually a List
accountList = {}  // It's a dictionary/map!

// Good
accountMap = {}
accountsByEmail = {}
accounts = {}
```

```pseudocode
// Bad - misleading similarity
XYZControllerForEfficientHandlingOfStrings
XYZControllerForEfficientStorageOfStrings

// Good - distinct names
StringProcessor
StringStorage
```

### 3. Make Meaningful Distinctions

If names must be different, they should mean something different.

```pseudocode
// Bad - noise words add no meaning
ProductInfo
ProductData
ProductObject

// Good - distinct meanings
Product          // The domain entity
ProductDTO       // Data transfer object
ProductViewModel // For display
```

```pseudocode
// Bad
getActiveAccount()
getActiveAccountInfo()
getActiveAccountData()

// Good - if they do different things, name them differently
getActiveAccount()        // Returns Account entity
getActiveAccountSummary() // Returns summary data
getActiveAccountDetails() // Returns full details
```

### 4. Use Pronounceable Names

If you can't pronounce it, you can't discuss it.

```pseudocode
// Bad
genymdhms  // Generation year, month, day, hour, minute, second
modymdhms

// Good
generationTimestamp
modificationTimestamp
```

### 5. Use Searchable Names

Single-letter names and numeric constants are hard to find.

```pseudocode
// Bad
for i in range(34) {
    s += t[i] * 4 / 5
}

// Good
WORK_DAYS_PER_WEEK = 5
HOURS_PER_DAY = 8

for taskIndex in range(numberOfTasks) {
    totalHours += taskHours[taskIndex] * HOURS_PER_DAY / WORK_DAYS_PER_WEEK
}
```

### 6. Avoid Encodings

Don't encode type or scope into names.

```pseudocode
// Bad - Hungarian notation
strName
intAge
bIsValid
m_description  // member prefix

// Good
name
age
isValid
description
```

### 7. Avoid Mental Mapping

Readers shouldn't have to mentally translate names.

```pseudocode
// Bad
for i in range(len(urls)) {
    u = urls[i]
    // Now reader must remember u = url
}

// Good
for url in urls {
    // Clear what we're working with
}
```

## Naming Conventions by Type

### Classes

Use nouns or noun phrases. Should represent a thing.

```pseudocode
// Good
Customer
WikiPage
Account
AddressParser

// Bad - verbs or vague
Manager    // Too generic
Processor  // What does it process?
Data       // What data?
Info       // What info?
```

### Methods/Functions

Use verbs or verb phrases. Should represent an action.

```pseudocode
// Good
save()
deletePage()
calculateTotal()
canEditUser()
isValidEmail()

// Accessors, mutators, predicates
getName()      // get prefix for accessors
setName(name)  // set prefix for mutators
isActive()     // is/has/can prefix for booleans
hasPermission()
canDelete()
```

### Variables

Use nouns that describe the content.

```pseudocode
// Good
userName
totalAmount
activeUsers
selectedProducts

// For booleans, use positive statements
isValid      // not isNotInvalid
hasChildren  // not doesNotHaveChildren
canEdit      // not cannotEdit
```

### Constants

Use all caps with underscores.

```pseudocode
MAX_CONNECTIONS = 100
DEFAULT_TIMEOUT = 30
API_BASE_URL = "https://api.example.com"
DAYS_PER_WEEK = 7
```

## Context Matters

### Add Meaningful Context

```pseudocode
// Bad - unclear without context
state  // State of what?

// Better - in context
class Address {
    state  // Clearly part of address
}

// Or add prefix when standalone
addressState
accountState
```

### Don't Add Gratuitous Context

```pseudocode
// Bad - redundant prefix
class GSDAccountAddress {
    GSDAccountAddressStreet
    GSDAccountAddressCity
    GSDAccountAddressState
}

// Good
class Address {
    street
    city
    state
}
```

## Domain-Specific Names

Use names from the problem domain or solution domain.

```pseudocode
// Problem domain (business terms)
customer
invoice
shipment
orderFulfillment

// Solution domain (technical terms)
queue
stack
factory
observer
visitor
```

Prefer problem domain names—they're what stakeholders understand.

## Common Name Patterns

### Collections

```pseudocode
// Plural for collections
users           // Array/List of users
userIds         // Array of IDs
productsByCategory  // Map keyed by category
```

### Booleans

```pseudocode
// Question-style names
isActive
hasPermission
canDelete
shouldRefresh
wasSuccessful
```

### Numbers

```pseudocode
// Include unit or purpose
timeoutSeconds
maxRetries
itemCount
pageNumber
distanceInMiles
```

### Pairs/Related Values

```pseudocode
// Consistent naming
startDate / endDate
minValue / maxValue
sourceFile / targetFile
oldPrice / newPrice
```

## Anti-Patterns

### Avoid These Names

```pseudocode
// Too short/cryptic
a, b, c, x, y (except loop counters or math)
tmp, temp
foo, bar
ret, result (unless returning is the sole purpose)

// Too generic
data
info
thing
object
manager
handler

// Misleading
list (when it's not a list)
array (when it's not an array)
```

## Refactoring for Better Names

When you find a bad name:

1. Understand what it really represents
2. Choose a name that reveals intent
3. Rename consistently throughout codebase
4. Update comments/docs if needed
5. Run tests to verify nothing broke

```pseudocode
// Before
function calc(a, b) {
    return a * b * 0.15
}

// After
function calculateTax(subtotal, taxRate) {
    return subtotal * taxRate
}
```

## Summary Checklist

- [ ] Does the name reveal intent?
- [ ] Is it pronounceable?
- [ ] Is it searchable?
- [ ] Does it avoid encodings?
- [ ] Is it appropriate length? (longer for larger scope)
- [ ] Does it use domain vocabulary?
- [ ] Is it consistent with similar names in codebase?

---

*Based on concepts from "Clean Code" by Robert C. Martin.*
