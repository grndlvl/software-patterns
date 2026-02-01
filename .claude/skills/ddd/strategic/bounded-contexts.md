# Bounded Contexts

## Definition

A **Bounded Context** is an explicit boundary within which a domain model exists. Inside this boundary, all terms, definitions, and rules have specific, unambiguous meaning. The same term can mean different things in different bounded contexts.

**Key Principle:** One model cannot accurately describe everything in a complex domain. Different parts of the system need different models, each valid within its own bounded context.

## Why Bounded Contexts Matter

### The Problem Without Boundaries

In a large system without explicit boundaries:

- **Linguistic Confusion:** The word "Customer" means different things to Sales, Support, and Billing
- **Model Corruption:** Trying to unify all meanings creates a bloated, compromised model
- **Integration Chaos:** Changes in one area unexpectedly break others
- **Team Conflicts:** Different teams argue over "correct" model definitions

### The Solution: Explicit Boundaries

Bounded Contexts solve these problems by:

1. **Linguistic Clarity:** Each context has its own ubiquitous language
2. **Model Integrity:** Each model remains pure and focused
3. **Team Autonomy:** Teams own their context and can evolve independently
4. **Explicit Integration:** Context boundaries make integration points clear

## Identifying Bounded Context Boundaries

### Team Boundaries

Teams naturally form linguistic boundaries:

- **Autonomous Teams:** Teams that work independently often have separate contexts
- **Communication Patterns:** Infrequent communication suggests different contexts
- **Ownership:** Clear ownership of code/functionality indicates boundaries

### Subdomain Boundaries

Business subdomains often align with bounded contexts:

- **Core Domain:** The competitive advantage (e.g., recommendation engine)
- **Supporting Subdomains:** Necessary but not differentiating (e.g., user management)
- **Generic Subdomains:** Common solutions (e.g., authentication, notifications)

### Language Boundaries

Listen for linguistic signals:

- **Overloaded Terms:** Same word, multiple meanings → separate contexts
- **Translation Required:** "In our terminology, their X means our Y" → boundary exists
- **Disambiguation Needed:** Constantly clarifying which meaning of a term → needs boundaries
- **Prefixing/Suffixing:** "Sales Customer" vs "Support Customer" → hidden contexts

### Model Boundaries

Model characteristics reveal boundaries:

- **Different Rules:** Customer validation differs between contexts
- **Different Lifecycles:** An entity created in one context, read-only in another
- **Different Relationships:** Product relates to Orders in Sales, to Inventory in Warehouse
- **Different Attributes:** Only some attributes matter in each context

## One Model Per Context

### The Single Model Rule

Within a bounded context:

- **One Ubiquitous Language:** All team members use identical terminology
- **One Consistent Model:** No contradictory definitions or rules
- **Clear Ownership:** One team, one vision, one model

### Example: "Customer" Across Contexts

The same real-world entity appears differently in different contexts:

**Sales Context:**
```pseudocode
class Customer {
    customerId: ID
    companyName: String
    industry: Industry
    accountManager: SalesRep
    leadSource: Source
    creditLimit: Money

    method canPlaceOrder(orderTotal: Money) -> Boolean {
        return orderTotal <= this.creditLimit
    }
}
```

**Support Context:**
```pseudocode
class Customer {
    customerId: ID
    contactName: String
    email: EmailAddress
    phone: PhoneNumber
    supportTier: Tier
    openTickets: List<Ticket>

    method canCreateTicket() -> Boolean {
        return this.supportTier.allowsNewTickets()
    }
}
```

**Billing Context:**
```pseudocode
class Customer {
    customerId: ID
    billingAddress: Address
    paymentMethod: PaymentMethod
    invoices: List<Invoice>
    accountBalance: Money

    method hasOutstandingBalance() -> Boolean {
        return this.accountBalance.isNegative()
    }
}
```

Same customer ID, completely different models—each valid within its context.

## Bounded Context vs Microservices

### They Are Not the Same

**Bounded Context:** A conceptual/linguistic boundary
**Microservice:** A deployment/physical boundary

### Relationship Options

**One Context, One Service:**
```pseudocode
// Ideal: Clear alignment
BillingContext -> BillingService (deployed independently)
```

**One Context, Multiple Services:**
```pseudocode
// Valid: Technical decomposition within a single model
OrderContext -> {
    OrderAPIService
    OrderProcessorService (background jobs)
    OrderReadModelService (CQRS read side)
}
```

**Multiple Contexts, One Service:**
```pseudocode
// Valid early on, but watch for growth
MonolithService -> {
    SalesContext (namespace/package)
    SupportContext (namespace/package)
    BillingContext (namespace/package)
}
```

**Anti-pattern: One Service, Multiple Contexts Mixed:**
```pseudocode
// Avoid: No boundaries, shared database
UserService -> {
    Sales models mixed with
    Support models mixed with
    Billing models
}
```

### Decision Factors

| Factor | Bounded Context | Microservice |
|--------|-----------------|--------------|
| **Primary Concern** | Model clarity, language | Deployment, scaling, independence |
| **Boundary Type** | Logical, conceptual | Physical, technical |
| **Driven By** | Domain complexity | Technical requirements |
| **Can Exist In** | Monolith or distributed | Must be distributed |
| **Team Alignment** | One team per context | Flexible |

**Guideline:** Start with bounded contexts in a monolith. Extract to microservices only when needed for scaling, team independence, or technical reasons.

## Physical vs Logical Boundaries

### Logical Boundaries (Always Required)

Exist even in a monolith:

```pseudocode
// Monolithic application with logical boundaries
package com.example.ecommerce {

    // Sales Context (logical boundary via package)
    package sales {
        class Customer { ... }
        class Order { ... }
        class Product { ... }
    }

    // Inventory Context (logical boundary via package)
    package inventory {
        class InventoryItem { ... }  // Different model than sales.Product
        class StockLevel { ... }
        class Warehouse { ... }
    }

    // Shipping Context (logical boundary via package)
    package shipping {
        class Shipment { ... }
        class ShippingAddress { ... }
    }
}
```

### Physical Boundaries (Optional)

Distributed deployment:

```pseudocode
// Sales Service (separate process/deployment)
service SalesService {
    database: sales_db
    api: /api/sales/*
    models: {
        Customer { customerId, companyName, creditLimit }
        Order { orderId, customerId, items[] }
    }
}

// Inventory Service (separate process/deployment)
service InventoryService {
    database: inventory_db
    api: /api/inventory/*
    models: {
        InventoryItem { sku, warehouseId, quantity }
        StockLevel { sku, available, reserved }
    }
}

// Communication via APIs (explicit integration)
class OrderService {
    method createOrder(order: Order) {
        // Check inventory in another service
        inventory = InventoryService.checkAvailability(order.items)
        if inventory.available {
            this.orderRepository.save(order)
            InventoryService.reserveStock(order.items)
        }
    }
}
```

### Boundary Enforcement

**In a Monolith:**
- Package/namespace structure
- Dependency rules (e.g., ArchUnit tests)
- Separate databases/schemas (optional)
- Access only via public APIs of each context

**In Microservices:**
- Network boundaries
- Separate deployments
- Separate databases (mandatory)
- Communication only via APIs/messages

## Examples: Different Models for Same Concept

### Example 1: Product

**Catalog Context (Search & Browse):**
```pseudocode
class Product {
    sku: SKU
    name: String
    description: RichText
    images: List<Image>
    category: Category
    searchKeywords: List<String>
    averageRating: Rating

    method isVisibleToCustomer() -> Boolean {
        return this.published and this.category.active
    }
}
```

**Inventory Context (Stock Management):**
```pseudocode
class InventoryItem {
    sku: SKU
    warehouseLocation: LocationCode
    quantityOnHand: Quantity
    quantityReserved: Quantity
    reorderPoint: Quantity

    method needsReorder() -> Boolean {
        available = quantityOnHand - quantityReserved
        return available < reorderPoint
    }
}
```

**Pricing Context (Dynamic Pricing):**
```pseudocode
class PricedProduct {
    sku: SKU
    basePrice: Money
    costOfGoods: Money
    competitorPrices: List<Money>
    priceElasticity: Decimal

    method calculateOptimalPrice(demand: Demand) -> Money {
        // Complex pricing algorithm
        return basePrice * demand.multiplier * priceElasticity
    }
}
```

### Example 2: User/Account

**Identity Context (Authentication):**
```pseudocode
class UserAccount {
    userId: UUID
    username: Username
    passwordHash: Hash
    email: Email
    mfaEnabled: Boolean

    method authenticate(password: String) -> Boolean {
        return passwordHash.verify(password)
    }
}
```

**Collaboration Context (Teamwork):**
```pseudocode
class TeamMember {
    memberId: UUID
    displayName: String
    avatarUrl: URL
    role: TeamRole
    teams: List<Team>

    method canAccessTeam(teamId: TeamId) -> Boolean {
        return teams.contains(teamId) and role.hasPermission()
    }
}
```

**Analytics Context (Usage Tracking):**
```pseudocode
class AnalyticsUser {
    anonymousId: Hash
    firstSeen: Timestamp
    lastSeen: Timestamp
    sessionCount: Integer
    features: List<FeatureUsage>

    method recordFeatureUsage(feature: Feature) {
        features.add(new FeatureUsage(feature, now()))
    }
}
```

### Example 3: Order

**Sales Context (Order Placement):**
```pseudocode
class Order {
    orderId: OrderId
    customerId: CustomerId
    items: List<LineItem>
    subtotal: Money
    tax: Money
    total: Money
    status: OrderStatus

    method addItem(product: Product, quantity: Quantity) {
        items.add(new LineItem(product, quantity))
        recalculateTotals()
    }
}
```

**Fulfillment Context (Picking & Shipping):**
```pseudocode
class ShippingOrder {
    orderId: OrderId
    destinationAddress: Address
    pickList: List<PickItem>
    priority: Priority
    carrier: Carrier

    method generatePackingSlip() -> PackingSlip {
        return new PackingSlip(pickList, destinationAddress)
    }
}
```

**Reporting Context (Business Intelligence):**
```pseudocode
class OrderSummary {
    orderId: OrderId
    orderDate: Date
    region: Region
    revenue: Money
    productCategories: List<Category>

    // Read-only, optimized for reporting
    // No business logic, just data
}
```

## Integration Between Contexts

### Context Map Patterns

**Shared Kernel (use sparingly):**
```pseudocode
// Shared between Sales and Billing contexts
module SharedCustomerIdentity {
    class CustomerId {
        value: UUID
    }
    class CustomerEmail {
        value: EmailAddress
    }
}
```

**Customer-Supplier (API dependency):**
```pseudocode
// Billing depends on Sales
class BillingService {
    method generateInvoice(customerId: CustomerId) {
        // Call Sales context API
        customer = SalesAPI.getCustomer(customerId)

        // Translate to Billing model
        billingCustomer = new BillingCustomer(
            customerId = customer.id,
            billingAddress = customer.defaultAddress
        )

        return createInvoice(billingCustomer)
    }
}
```

**Published Language (standardized format):**
```pseudocode
// Event published by Sales, consumed by multiple contexts
event OrderPlaced {
    orderId: UUID
    customerId: UUID
    items: [{
        sku: String
        quantity: Integer
        price: Decimal
    }]
    total: Decimal
    timestamp: ISO8601
}

// Inventory consumes
class InventoryEventHandler {
    method onOrderPlaced(event: OrderPlaced) {
        foreach item in event.items {
            reserveStock(item.sku, item.quantity)
        }
    }
}

// Shipping consumes
class ShippingEventHandler {
    method onOrderPlaced(event: OrderPlaced) {
        createShipment(event.orderId, event.items)
    }
}
```

## Summary Table

| Aspect | Description |
|--------|-------------|
| **Purpose** | Explicit boundary for a unified model and language |
| **Size** | As large as can maintain model consistency |
| **Team Alignment** | Ideally one team owns one context |
| **Language** | One ubiquitous language per context |
| **Model** | One consistent model, no contradictions |
| **Integration** | Explicit via APIs, events, or shared kernel |
| **Physical Boundary** | Optional; can exist in monolith or as microservice |
| **Subdomain Alignment** | Often aligns, but not required |
| **Identification Signals** | Overloaded terms, team boundaries, different rules |
| **Anti-pattern** | One model trying to serve multiple contexts |
| **Key Benefit** | Model clarity, team autonomy, controlled complexity |

---

*Based on concepts from "Domain-Driven Design" by Eric Evans.*
