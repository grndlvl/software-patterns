# Reversibility

## Definition

> "There are no final decisions."
> — Dave Thomas & Andy Hunt

Nothing is forever. Requirements change, technology evolves, and today's perfect solution becomes tomorrow's legacy burden. Design for change.

## The Problem with Finality

When you make an "irreversible" decision:
- You bet on predictions being correct
- You lock in technical debt
- You can't adapt to new requirements
- Migration becomes expensive or impossible

## Critical Decisions to Keep Reversible

### Database Choice

```pseudocode
// Bad - hardcoded to PostgreSQL
class UserRepository {
    function findById(id) {
        return pg.query("SELECT * FROM users WHERE id = $1", [id])
    }
}

// Good - abstracted
interface UserRepository {
    function findById(id): User
}

class PostgresUserRepository implements UserRepository {
    function findById(id) {
        return pg.query("SELECT * FROM users WHERE id = $1", [id])
    }
}

class MongoUserRepository implements UserRepository {
    function findById(id) {
        return mongo.collection("users").findOne({ _id: id })
    }
}

// Now switching databases is straightforward
```

### Third-Party Services

```pseudocode
// Bad - hardcoded to specific provider
function sendEmail(to, subject, body) {
    sendgrid.send({
        to: to,
        from: "noreply@example.com",
        subject: subject,
        html: body
    })
}

// Good - abstracted
interface EmailService {
    function send(message: EmailMessage)
}

class SendGridEmailService implements EmailService {
    function send(message) { /* SendGrid API */ }
}

class SESEmailService implements EmailService {
    function send(message) { /* AWS SES API */ }
}

class MailgunEmailService implements EmailService {
    function send(message) { /* Mailgun API */ }
}

// Switch providers by changing configuration
```

### Architecture Patterns

```pseudocode
// Bad - monolith assumptions everywhere
function processOrder(order) {
    // Directly calls inventory
    inventory.reserve(order.items)

    // Directly calls payment
    payment.charge(order.total)

    // Directly calls shipping
    shipping.createLabel(order)
}

// Good - message-based (can split into microservices later)
function processOrder(order) {
    eventBus.publish("OrderPlaced", {
        orderId: order.id,
        items: order.items,
        total: order.total
    })
}

// Handlers can be in same process or different services
eventBus.subscribe("OrderPlaced", inventoryHandler)
eventBus.subscribe("OrderPlaced", paymentHandler)
eventBus.subscribe("OrderPlaced", shippingHandler)
```

## Strategies for Reversibility

### 1. Hide Third-Party APIs Behind Abstractions

```pseudocode
// Wrap every external service
interface PaymentGateway {
    function charge(amount, card): PaymentResult
    function refund(transactionId): RefundResult
}

// Implementation details hidden
class StripeGateway implements PaymentGateway { }
class BraintreeGateway implements PaymentGateway { }
class PayPalGateway implements PaymentGateway { }
```

### 2. Use Configuration, Not Code

```pseudocode
// Bad - hardcoded
DATABASE_HOST = "prod-db-1.example.com"
API_URL = "https://api.vendor.com/v2"

// Good - configurable
DATABASE_HOST = env("DATABASE_HOST")
API_URL = env("API_URL")

// Can change without redeploying
```

### 3. Design for Replacement

Ask: "How hard would it be to replace this component?"

```pseudocode
// Bad - framework deeply embedded
class User extends FrameworkActiveRecord {
    // All logic tied to framework
}

// Good - domain logic separated
class User {
    // Pure domain logic, no framework dependencies
}

class UserActiveRecord extends FrameworkActiveRecord {
    // Framework adapter
    function toUser() {
        return new User(this.attributes)
    }
}
```

### 4. Keep Components Small

Small components are easier to replace than large ones.

```
// Bad: One big payment module
PaymentSystem (10,000 lines)

// Good: Small, replaceable pieces
CardValidator (200 lines)
FraudChecker (300 lines)
PaymentProcessor (400 lines)
ReceiptGenerator (250 lines)
```

### 5. Write Reversible Migrations

```pseudocode
// Database migration
function up() {
    addColumn("users", "phone", "varchar(20)")
}

function down() {
    removeColumn("users", "phone")  // Can undo!
}

// Data migration
function up() {
    // Save old data before transforming
    backup = database.query("SELECT * FROM users")
    saveBackup(backup)

    // Transform
    transformData()
}

function down() {
    restoreBackup()  // Can undo!
}
```

## Real-World Reversibility

### Vendor Lock-In

| Area | Locked In | Reversible |
|------|-----------|------------|
| Cloud | Using AWS-specific services everywhere | Cloud-agnostic abstractions |
| Database | Stored procedures, proprietary features | Standard SQL, repository pattern |
| Framework | Business logic in framework controllers | Framework as thin adapter |
| Auth | Proprietary user format | Standard claims/tokens |

### Technology Bets

| Bet | Hedged Version |
|-----|----------------|
| "GraphQL is the future" | REST + GraphQL adapter |
| "NoSQL will scale better" | Repository pattern (swap later) |
| "Kubernetes forever" | Container-agnostic deployment |
| "React won" | Component abstraction layer |

## The Flexible Architecture

```
┌─────────────────────────────────────────────────┐
│                   UI Layer                       │
│  (Can switch: React → Vue → Server-rendered)    │
├─────────────────────────────────────────────────┤
│                  API Layer                       │
│  (Can switch: REST → GraphQL → gRPC)            │
├─────────────────────────────────────────────────┤
│              Business Logic                      │
│  (Pure domain code, no external dependencies)   │
├─────────────────────────────────────────────────┤
│           Infrastructure Adapters                │
│  (Can switch: Postgres → MySQL → DynamoDB)      │
│  (Can switch: Stripe → Braintree → PayPal)      │
└─────────────────────────────────────────────────┘
```

## Signs of Irreversibility

🚩 **Red flags:**
- "We're committed to X vendor"
- "Switching would require a rewrite"
- "It's embedded throughout the codebase"
- "We use proprietary features heavily"
- "Our data format only works with X"

✅ **Good signs:**
- "We could switch in a few weeks"
- "It's behind an interface"
- "We use standard formats"
- "The business logic doesn't know about infrastructure"

## Summary

> "The mistake lies in assuming that any decision is cast in stone—and in not preparing for the contingencies that might arise."

| Do | Don't |
|----|-------|
| Abstract external dependencies | Call vendor APIs directly |
| Use configuration | Hardcode values |
| Design small, replaceable parts | Build monolithic components |
| Use standard formats | Use proprietary formats |
| Write reversible migrations | Make one-way changes |
| Question "best" solutions | Assume today's choice is forever |

---

*Based on concepts from "The Pragmatic Programmer" by David Thomas and Andrew Hunt.*
