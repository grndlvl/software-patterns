# Interface Segregation Principle (ISP)

## Definition

> "Clients should not be forced to depend on interfaces they do not use."
> — Robert C. Martin

Many specific interfaces are better than one general-purpose interface. No client should be forced to implement methods it doesn't need.

## The Problem

When interfaces are too large:
- Implementers must provide methods they don't need
- Changes to unused methods still affect implementers
- Classes become coupled to features they don't use
- Testing requires mocking unnecessary methods

## Recognizing Violations

**Warning signs:**
- Interface has many methods but implementers only use a few
- Empty or throwing implementations of interface methods
- "Fat" interfaces that do many unrelated things
- Implementing class has methods that don't make sense for it

## Examples

### Bad: Fat Interface

```pseudocode
interface Worker {
    function work()
    function eat()
    function sleep()
    function attendMeeting()
    function writeReport()
    function reviewCode()
}

class Developer implements Worker {
    function work() { /* coding */ }
    function eat() { /* lunch */ }
    function sleep() { /* N/A at work? */ }
    function attendMeeting() { /* ... */ }
    function writeReport() { /* maybe */ }
    function reviewCode() { /* yes */ }
}

class Robot implements Worker {
    function work() { /* processing */ }
    function eat() { throw NotApplicable }      // Forced to implement!
    function sleep() { throw NotApplicable }    // Forced to implement!
    function attendMeeting() { throw NotApplicable }
    function writeReport() { /* can generate */ }
    function reviewCode() { /* can lint */ }
}
```

### Good: Segregated Interfaces

```pseudocode
interface Workable {
    function work()
}

interface Feedable {
    function eat()
}

interface Restable {
    function sleep()
}

interface Meetable {
    function attendMeeting()
}

interface Reporter {
    function writeReport()
}

interface CodeReviewer {
    function reviewCode()
}

class Developer implements Workable, Feedable, Meetable, CodeReviewer {
    function work() { /* coding */ }
    function eat() { /* lunch */ }
    function attendMeeting() { /* ... */ }
    function reviewCode() { /* yes */ }
}

class Robot implements Workable, Reporter, CodeReviewer {
    function work() { /* processing */ }
    function writeReport() { /* can generate */ }
    function reviewCode() { /* can lint */ }
}

// Each class only implements what it needs!
```

### Bad: Printer Interface

```pseudocode
interface MultiFunctionDevice {
    function print(document)
    function scan(document)
    function fax(document)
    function staple(document)
}

class BasicPrinter implements MultiFunctionDevice {
    function print(document) { /* OK */ }
    function scan(document) { throw NotSupported }  // Can't scan!
    function fax(document) { throw NotSupported }   // Can't fax!
    function staple(document) { throw NotSupported } // Can't staple!
}
```

### Good: Role Interfaces

```pseudocode
interface Printer {
    function print(document)
}

interface Scanner {
    function scan(document)
}

interface Fax {
    function fax(document)
}

interface Stapler {
    function staple(document)
}

class BasicPrinter implements Printer {
    function print(document) { /* printing logic */ }
}

class AllInOnePrinter implements Printer, Scanner, Fax {
    function print(document) { /* ... */ }
    function scan(document) { /* ... */ }
    function fax(document) { /* ... */ }
}

class OfficePrinter implements Printer, Scanner, Stapler {
    function print(document) { /* ... */ }
    function scan(document) { /* ... */ }
    function staple(document) { /* ... */ }
}
```

### Bad: Repository Interface

```pseudocode
interface Repository {
    function findById(id)
    function findAll()
    function findByName(name)
    function findByEmail(email)
    function findByDateRange(start, end)
    function save(entity)
    function update(entity)
    function delete(entity)
    function bulkInsert(entities)
    function executeRawQuery(sql)
}

// ProductRepository doesn't need findByEmail!
// UserRepository doesn't need findByDateRange!
```

### Good: Composed Interfaces

```pseudocode
interface ReadRepository<T> {
    function findById(id): T
    function findAll(): List<T>
}

interface WriteRepository<T> {
    function save(entity: T)
    function delete(entity: T)
}

interface Repository<T> extends ReadRepository<T>, WriteRepository<T> { }

// Specific query interfaces
interface UserQueries {
    function findByEmail(email): User
}

interface ProductQueries {
    function findByCategory(category): List<Product>
}

class UserRepository implements Repository<User>, UserQueries {
    // Only implements what makes sense for users
}

class ProductRepository implements Repository<Product>, ProductQueries {
    // Only implements what makes sense for products
}

// Read-only repository for reports
class ReportingRepository implements ReadRepository<SalesData> {
    // No write methods needed
}
```

## Client-Specific Interfaces

Design interfaces from the client's perspective:

```pseudocode
// What does the client need?

// Authentication module needs:
interface Authenticatable {
    function getCredentials()
    function validatePassword(password)
}

// Display module needs:
interface Displayable {
    function getName()
    function getAvatar()
}

// Notification module needs:
interface Notifiable {
    function getEmail()
    function getNotificationPreferences()
}

// User implements all relevant interfaces
class User implements Authenticatable, Displayable, Notifiable {
    // Implements all methods, but each client only sees what it needs
}

// Authentication service only depends on Authenticatable
class AuthService {
    function login(user: Authenticatable, password) {
        return user.validatePassword(password)
    }
}

// Display component only depends on Displayable
class UserCard {
    function render(user: Displayable) {
        return "<div>{user.getName()}<img src='{user.getAvatar()}'></div>"
    }
}
```

## Interface Composition

Combine small interfaces as needed:

```pseudocode
interface Identifiable {
    function getId()
}

interface Timestamped {
    function getCreatedAt()
    function getUpdatedAt()
}

interface SoftDeletable {
    function delete()
    function restore()
    function isDeleted()
}

// Compose for specific needs
interface Entity extends Identifiable, Timestamped { }

interface DeletableEntity extends Entity, SoftDeletable { }

// Minimal implementation
class SimpleEntity implements Identifiable {
    id
    function getId() { return id }
}

// Full-featured implementation
class AuditedEntity implements DeletableEntity {
    id
    createdAt
    updatedAt
    deletedAt

    function getId() { return id }
    function getCreatedAt() { return createdAt }
    function getUpdatedAt() { return updatedAt }
    function delete() { deletedAt = now() }
    function restore() { deletedAt = null }
    function isDeleted() { return deletedAt != null }
}
```

## Signs of Good Segregation

- Interfaces have 1-5 methods (cohesive)
- Implementing classes use ALL methods
- No empty or throwing implementations
- Interfaces named for roles, not implementations
- Easy to mock for testing

## Benefits

- **Decoupling**: Clients only depend on what they use
- **Flexibility**: Easy to add new implementations
- **Testability**: Small interfaces are easy to mock
- **Clarity**: Interfaces reveal intent
- **Maintainability**: Changes have limited scope

## Related Principles

- **Single Responsibility**: Applied to interfaces
- **Liskov Substitution**: Small interfaces are easier to implement correctly
- **Dependency Inversion**: Depend on focused abstractions

---

*Based on concepts from "Clean Code" and "Agile Software Development" by Robert C. Martin.*
