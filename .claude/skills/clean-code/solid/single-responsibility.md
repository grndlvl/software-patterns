# Single Responsibility Principle (SRP)

## Definition

> "A class should have one, and only one, reason to change."
> — Robert C. Martin

A class should have only one responsibility—one job to do. If a class has more than one responsibility, those responsibilities become coupled, and changes to one may affect the other.

## The Problem

When a class has multiple responsibilities:
- Changes for one reason may break unrelated functionality
- The class becomes harder to understand
- Testing becomes more complex
- Reuse becomes difficult

## Recognizing Violations

**Warning signs:**
- Class name includes "And" or "Or"
- Class has methods that don't use the same instance variables
- Changes to one feature require modifying the same class as another feature
- Class description requires the word "and"

## Examples

### Bad: Multiple Responsibilities

```pseudocode
class Employee {
    name
    email
    salary

    function calculatePay() {
        // Payroll calculation logic
        return salary * hoursWorked * taxRate
    }

    function saveToDatabase() {
        // Persistence logic
        db.execute("INSERT INTO employees...")
    }

    function generateReport() {
        // Reporting logic
        return formatAsHTML(this)
    }
}

// This class has THREE reasons to change:
// 1. Pay calculation rules change
// 2. Database schema changes
// 3. Report format changes
```

### Good: Separated Responsibilities

```pseudocode
class Employee {
    name
    email
    salary
}

class PayCalculator {
    function calculatePay(employee) {
        return employee.salary * hoursWorked * taxRate
    }
}

class EmployeeRepository {
    function save(employee) {
        db.execute("INSERT INTO employees...")
    }

    function findById(id) {
        return db.query("SELECT * FROM employees WHERE id = ?", id)
    }
}

class EmployeeReportGenerator {
    function generateReport(employee) {
        return formatAsHTML(employee)
    }
}

// Now each class has ONE reason to change
```

### Bad: God Class

```pseudocode
class UserManager {
    function createUser(data) { /* ... */ }
    function deleteUser(id) { /* ... */ }
    function validateEmail(email) { /* ... */ }
    function sendWelcomeEmail(user) { /* ... */ }
    function hashPassword(password) { /* ... */ }
    function generateAuthToken(user) { /* ... */ }
    function logUserActivity(user, action) { /* ... */ }
    function exportUsersToCsv() { /* ... */ }
}

// This "God class" handles: CRUD, validation, email,
// security, authentication, logging, and reporting
```

### Good: Focused Classes

```pseudocode
class UserService {
    repository
    validator

    function createUser(data) {
        validator.validate(data)
        user = new User(data)
        repository.save(user)
        return user
    }
}

class EmailValidator {
    function validate(email) {
        return regex.match(emailPattern, email)
    }
}

class WelcomeEmailSender {
    function send(user) {
        mailer.send(user.email, welcomeTemplate)
    }
}

class PasswordHasher {
    function hash(password) {
        return bcrypt.hash(password)
    }
}

class AuthTokenGenerator {
    function generate(user) {
        return jwt.sign(user.id, secret)
    }
}
```

## Applying SRP

### Step 1: Identify Responsibilities

List what the class does:
- "This class calculates pay AND saves to database AND generates reports"

### Step 2: Group by Actor

Who requests changes to each responsibility?
- Pay calculation → Accounting department
- Database → DBAs / Tech team
- Reports → Management

Different actors = different responsibilities = separate classes

### Step 3: Extract Classes

Create focused classes for each responsibility:
```pseudocode
// Before: One class, three responsibilities
class Employee { /* everything */ }

// After: Three classes, one responsibility each
class Employee { /* data only */ }
class PayrollService { /* pay calculation */ }
class EmployeeRepository { /* persistence */ }
```

## SRP at Different Levels

### Function Level
```pseudocode
// Bad: Function does two things
function validateAndSave(user) {
    if not isValid(user) {
        throw ValidationError
    }
    database.save(user)
}

// Good: Separate functions
function validate(user) {
    if not isValid(user) {
        throw ValidationError
    }
}

function save(user) {
    database.save(user)
}
```

### Module/Package Level
```
// Bad: One module for everything
src/
  user.php  // CRUD, validation, email, auth, reports

// Good: Separate modules
src/
  user/
    User.php
    UserRepository.php
  validation/
    EmailValidator.php
  auth/
    TokenGenerator.php
  email/
    WelcomeMailer.php
```

## Common Misconceptions

### "One Method Per Class"
SRP doesn't mean one method per class. It means one *responsibility*—which may require multiple methods.

```pseudocode
// This is fine - all methods serve one responsibility (persistence)
class UserRepository {
    function save(user) { }
    function findById(id) { }
    function findByEmail(email) { }
    function delete(user) { }
}
```

### "Any Change = Violation"
Not every change means a violation. The question is: do changes come from different *actors* or *reasons*?

## Benefits

- **Easier testing**: Smaller, focused classes are easier to test
- **Lower coupling**: Changes don't ripple through unrelated code
- **Better reuse**: Focused classes can be reused in different contexts
- **Clearer intent**: Class names reveal exactly what they do
- **Parallel development**: Teams can work on different responsibilities independently

## Related Principles

- **Interface Segregation**: SRP applied to interfaces
- **Separation of Concerns**: Broader architectural principle
- **Cohesion**: High cohesion = single responsibility

---

*Based on concepts from "Clean Code" and "Agile Software Development" by Robert C. Martin.*
