# Context Mapping

## Definition

A **Context Map** is a strategic design pattern that documents the relationships and integration patterns between different Bounded Contexts. It provides a high-level view of the system's architecture, showing how different contexts interact, what their relationship dynamics are, and what integration patterns are used.

The Context Map serves as:
- A communication tool between teams
- A strategic planning artifact
- A guide for integration design
- Documentation of organizational and technical boundaries

## Mapping Relationships

### Partnership

**Definition:** Two contexts with dependent features where teams collaborate closely to achieve shared goals.

**Characteristics:**
- Mutual dependency between contexts
- Joint planning and coordination
- Shared success or failure
- Coordinated releases

**When to Use:**
- Teams have aligned goals
- Features span multiple contexts
- High degree of trust between teams
- Need for synchronized evolution

```pseudocode
// E-commerce and Inventory contexts in partnership
CONTEXT ECommerce
  COLLABORATES_WITH Inventory

  OPERATION placeOrder(order)
    // Coordinate with inventory context
    IF Inventory.reserveItems(order.items) THEN
      processPayment(order)
      Inventory.commitReservation(order.id)
    ELSE
      RETURN "Items unavailable"
    END
  END
END

CONTEXT Inventory
  COLLABORATES_WITH ECommerce

  OPERATION reserveItems(items)
    // Joint protocol designed together
    RETURN checkAndHoldStock(items)
  END

  OPERATION commitReservation(orderId)
    finalizeStockAllocation(orderId)
  END
END
```

---

### Shared Kernel

**Definition:** A subset of the domain model shared between two or more contexts, requiring careful coordination to modify.

**Characteristics:**
- Common code shared by multiple contexts
- Changes require agreement from all teams
- Typically includes core domain concepts
- High coupling but intentional

**When to Use:**
- Small, stable subset can be shared
- Cost of duplication exceeds coordination cost
- Strong need for consistency
- Teams can coordinate effectively

```pseudocode
// Shared kernel between Order and Billing contexts
MODULE SharedKernel
  // Shared value objects
  CLASS Money
    PROPERTIES
      amount: Decimal
      currency: String

    METHOD add(other: Money): Money
      ASSERT this.currency = other.currency
      RETURN Money(this.amount + other.amount, this.currency)
    END
  END

  CLASS CustomerId
    PROPERTIES
      value: String

    METHOD validate(): Boolean
      RETURN isValidFormat(this.value)
    END
  END
END

CONTEXT Orders
  USES SharedKernel

  CLASS Order
    PROPERTIES
      total: SharedKernel.Money
      customerId: SharedKernel.CustomerId
  END
END

CONTEXT Billing
  USES SharedKernel

  CLASS Invoice
    PROPERTIES
      amount: SharedKernel.Money
      customerId: SharedKernel.CustomerId
  END
END
```

---

### Customer-Supplier

**Definition:** Upstream context (supplier) provides services to downstream context (customer), with downstream team able to negotiate requirements.

**Characteristics:**
- Clear dependency direction
- Customer can influence supplier's roadmap
- Formal planning and prioritization
- Service Level Agreements (SLAs)

**When to Use:**
- Clear upstream/downstream relationship
- Customer has influence over supplier
- Need for negotiated features
- Formal development process

```pseudocode
// Product Catalog (supplier) and Storefront (customer)
CONTEXT ProductCatalog // UPSTREAM
  PUBLISHED_INTERFACE
    OPERATION getProduct(productId): Product
    OPERATION searchProducts(criteria): List<Product>
    OPERATION getProductAvailability(productId): Availability
  END

  // Customer team can request new features
  ROADMAP
    ITEM "Add bulk product lookup" PRIORITY High REQUESTED_BY Storefront
    ITEM "Include shipping dimensions" PRIORITY Medium REQUESTED_BY Logistics
  END
END

CONTEXT Storefront // DOWNSTREAM
  CONSUMES ProductCatalog

  OPERATION displayProductPage(productId)
    product = ProductCatalog.getProduct(productId)
    availability = ProductCatalog.getProductAvailability(productId)

    render(product, availability)
  END

  // Can request features but supplier decides priority
  REQUEST_FEATURE "real-time inventory count" TO ProductCatalog
END
```

---

### Conformist

**Definition:** Downstream context conforms to the upstream context's model without influence or translation.

**Characteristics:**
- No negotiation power
- Must accept upstream model as-is
- No translation layer
- Simple integration

**When to Use:**
- Upstream team won't accommodate changes
- Cost of influence exceeds benefit
- Upstream model is good enough
- External/third-party dependency

```pseudocode
// Internal system conforming to external payment gateway
CONTEXT PaymentProcessing
  CONFORMS_TO StripePaymentGateway // EXTERNAL

  // Directly use Stripe's model and terminology
  CLASS Payment
    PROPERTIES
      // Using Stripe's exact structure
      amount: Integer // cents, not dollars
      currency: String // ISO code
      paymentMethodId: String // Stripe's identifier
      metadata: Map<String, String>
  END

  OPERATION processPayment(payment: Payment)
    // No translation - pass directly to Stripe
    result = StripeAPI.createPaymentIntent(
      amount: payment.amount,
      currency: payment.currency,
      payment_method: payment.paymentMethodId,
      metadata: payment.metadata
    )

    RETURN result
  END
END

EXTERNAL_CONTEXT StripePaymentGateway
  // We have no influence over this model
  // We must conform to their API and terminology
END
```

---

### Anti-Corruption Layer (ACL)

**Definition:** A translation layer that isolates a downstream context from the model and interface of an upstream context.

**Characteristics:**
- Protects domain model integrity
- Translates between models
- Insulates from upstream changes
- Additional complexity but clean separation

**When to Use:**
- Upstream model doesn't fit your domain
- Need to protect your model's integrity
- Upstream is legacy or poorly designed
- Multiple upstream dependencies

```pseudocode
// ACL protecting modern system from legacy database
CONTEXT OrderManagement
  PROTECTED_BY OrderLegacyACL

  // Clean domain model
  CLASS Order
    PROPERTIES
      orderId: OrderId
      customer: Customer
      items: List<OrderLine>
      total: Money
      status: OrderStatus

    METHOD calculateTotal(): Money
      RETURN items.sum(line => line.subtotal())
    END
  END

  ENUM OrderStatus
    PENDING, CONFIRMED, SHIPPED, DELIVERED, CANCELLED
  END
END

MODULE OrderLegacyACL // Anti-Corruption Layer
  // Translates between clean domain and legacy format

  CLASS LegacyOrderAdapter
    METHOD toDomain(legacyData): Order
      // Complex translation logic
      status = SWITCH legacyData.stat_cd
        CASE "P" => OrderStatus.PENDING
        CASE "C" => OrderStatus.CONFIRMED
        CASE "S" => OrderStatus.SHIPPED
        CASE "D" => OrderStatus.DELIVERED
        CASE "X" => OrderStatus.CANCELLED
      END

      items = legacyData.ord_lines.map(line =>
        OrderLine(
          productId: ProductId(line.prod_id),
          quantity: line.qty,
          price: Money(line.amt / 100, "USD")
        )
      )

      RETURN Order(
        orderId: OrderId(legacyData.ord_num),
        customer: Customer(legacyData.cust_id),
        items: items,
        status: status
      )
    END

    METHOD toLegacy(order: Order): LegacyOrderRecord
      // Reverse translation
      RETURN LegacyOrderRecord(
        ord_num: order.orderId.value,
        cust_id: order.customer.id.value,
        stat_cd: statusToCode(order.status),
        ord_lines: order.items.map(toLegacyLine),
        tot_amt: order.total.amount * 100
      )
    END
  END
END

LEGACY_CONTEXT OrderDatabase
  // Ugly legacy model we're protecting against
  // Uses cryptic codes, cents instead of Money objects, etc.
END
```

---

### Open Host Service

**Definition:** A protocol that provides access to a context as a set of services, designed to meet the needs of multiple clients.

**Characteristics:**
- Designed for multiple consumers
- Stable, published interface
- Often combined with Published Language
- Versioned API

**When to Use:**
- Multiple downstream contexts
- Need for stable integration points
- Want to encapsulate internal complexity
- Standard integration pattern needed

```pseudocode
// Product Catalog as Open Host Service
CONTEXT ProductCatalog
  OPEN_HOST_SERVICE ProductCatalogAPI
    VERSION "2.0"

    // RESTful API designed for multiple consumers
    ENDPOINT GET "/api/v2/products/{productId}"
      RETURNS ProductDTO
      STATUS_CODES 200, 404
    END

    ENDPOINT GET "/api/v2/products"
      PARAMETERS
        category: String OPTIONAL
        minPrice: Decimal OPTIONAL
        maxPrice: Decimal OPTIONAL
        page: Integer DEFAULT 1
        pageSize: Integer DEFAULT 20
      RETURNS PagedResult<ProductDTO>
    END

    ENDPOINT POST "/api/v2/products"
      REQUIRES_AUTHENTICATION
      REQUIRES_ROLE "PRODUCT_MANAGER"
      BODY CreateProductRequest
      RETURNS ProductDTO
      STATUS_CODES 201, 400, 401, 403
    END

    // Data Transfer Objects - Published Language
    DTO ProductDTO
      PROPERTIES
        id: String
        name: String
        description: String
        price: PriceDTO
        category: String
        availability: String // "IN_STOCK" | "LOW_STOCK" | "OUT_OF_STOCK"
        imageUrls: List<String>
    END

    DTO PriceDTO
      PROPERTIES
        amount: Decimal
        currency: String // ISO 4217
    END
  END
END

// Multiple contexts can consume the Open Host Service
CONTEXT Storefront
  CONSUMES ProductCatalog.ProductCatalogAPI
END

CONTEXT RecommendationEngine
  CONSUMES ProductCatalog.ProductCatalogAPI
END

CONTEXT PriceComparison
  CONSUMES ProductCatalog.ProductCatalogAPI
END
```

---

### Published Language

**Definition:** A well-documented, shared language for communication between contexts, often in the form of a data interchange format.

**Characteristics:**
- Standardized format (XML, JSON, Protocol Buffers)
- Comprehensive documentation
- Versioned schema
- Independent of implementation

**When to Use:**
- Need for interoperability
- Multiple integrations
- Long-term stability required
- Industry standards exist

```pseudocode
// Order event Published Language using JSON Schema
MODULE OrderEventLanguage
  SCHEMA "OrderPlacedEvent" VERSION "1.0"
    DESCRIPTION "Published when a customer places an order"

    STRUCTURE
      eventId: UUID REQUIRED
      eventType: "OrderPlaced" REQUIRED
      timestamp: ISO8601DateTime REQUIRED
      version: "1.0" REQUIRED

      payload:
        orderId: String REQUIRED PATTERN "^ORD-[0-9]{8}$"
        customerId: String REQUIRED
        orderDate: ISO8601DateTime REQUIRED

        items: Array REQUIRED MIN_ITEMS 1
          EACH_ITEM:
            productId: String REQUIRED
            productName: String REQUIRED
            quantity: Integer REQUIRED MIN 1
            unitPrice:
              amount: Decimal REQUIRED MIN 0
              currency: String REQUIRED PATTERN "^[A-Z]{3}$"

        shippingAddress:
          street: String REQUIRED
          city: String REQUIRED
          postalCode: String REQUIRED
          country: String REQUIRED PATTERN "^[A-Z]{2}$"

        totalAmount:
          amount: Decimal REQUIRED MIN 0
          currency: String REQUIRED PATTERN "^[A-Z]{3}$"
    END
  END

  EXAMPLE
    {
      "eventId": "550e8400-e29b-41d4-a716-446655440000",
      "eventType": "OrderPlaced",
      "timestamp": "2024-01-15T14:30:00Z",
      "version": "1.0",
      "payload": {
        "orderId": "ORD-12345678",
        "customerId": "CUST-98765",
        "orderDate": "2024-01-15T14:30:00Z",
        "items": [
          {
            "productId": "PROD-001",
            "productName": "Widget",
            "quantity": 2,
            "unitPrice": {
              "amount": 29.99,
              "currency": "USD"
            }
          }
        ],
        "shippingAddress": {
          "street": "123 Main St",
          "city": "Springfield",
          "postalCode": "12345",
          "country": "US"
        },
        "totalAmount": {
          "amount": 59.98,
          "currency": "USD"
        }
      }
    }
  END
END

// Contexts use the Published Language
CONTEXT OrderProcessing
  PUBLISHES OrderEventLanguage.OrderPlacedEvent

  OPERATION publishOrderPlaced(order: Order)
    event = createEvent(OrderEventLanguage.OrderPlacedEvent, order)
    eventBus.publish(event)
  END
END

CONTEXT Fulfillment
  SUBSCRIBES_TO OrderEventLanguage.OrderPlacedEvent

  OPERATION handleOrderPlaced(event)
    // Parse according to published schema
    order = parseOrderEvent(event)
    createShipment(order)
  END
END
```

---

### Separate Ways

**Definition:** Two contexts have no connection or integration; each solves its own problems independently.

**Characteristics:**
- No integration
- Duplicate functionality acceptable
- Complete independence
- Different solutions to similar problems

**When to Use:**
- Integration cost exceeds benefit
- Different priorities or timelines
- Temporary separation
- Truly independent domains

```pseudocode
// Marketing and Engineering both track "users" separately
CONTEXT MarketingCampaigns
  // Marketing's view of users
  CLASS MarketingContact
    PROPERTIES
      email: String
      firstName: String
      lastName: String
      segments: List<String>
      lastEmailSent: DateTime
      engagementScore: Integer
      preferences: MarketingPreferences

    METHOD calculateEngagementScore(): Integer
      // Marketing-specific logic
    END
  END

  // Marketing manages their own database
  DATABASE marketing_db
    TABLE contacts
    TABLE campaigns
    TABLE email_sends
  END
END

CONTEXT UserAuthentication
  // Engineering's view of users
  CLASS User
    PROPERTIES
      userId: UUID
      username: String
      email: String
      passwordHash: String
      roles: List<Role>
      lastLogin: DateTime
      mfaEnabled: Boolean

    METHOD authenticate(password): Boolean
      // Authentication-specific logic
    END
  END

  // Engineering manages their own database
  DATABASE app_db
    TABLE users
    TABLE sessions
    TABLE permissions
  END
END

// No synchronization, no shared model
// Email might be duplicated but that's acceptable
// Each context solves its own problems
```

---

## Creating a Context Map

### Step 1: Identify Bounded Contexts

```pseudocode
IDENTIFY_CONTEXTS
  1. List all major subsystems
  2. Identify distinct models
  3. Find team boundaries
  4. Recognize language shifts

EXAMPLE:
  - Order Management
  - Inventory
  - Shipping
  - Customer Service
  - Billing
  - Analytics
END
```

### Step 2: Identify Relationships

```pseudocode
FOR EACH pair of contexts
  ASK:
    - Does one depend on the other?
    - Do they share code or data?
    - How do teams interact?
    - What integration patterns exist?

  DETERMINE relationship type
END
```

### Step 3: Document Dependencies

```pseudocode
CONTEXT_MAP
  CONTEXT OrderManagement
    DOWNSTREAM_FROM Inventory (Customer-Supplier)
    DOWNSTREAM_FROM PaymentGateway (Conformist)
    PARTNER_WITH Shipping
    SHARES_KERNEL_WITH Billing
  END

  CONTEXT Inventory
    UPSTREAM_TO OrderManagement
    UPSTREAM_TO Shipping
    OPEN_HOST ProductCatalog API
  END

  CONTEXT PaymentGateway (EXTERNAL)
    UPSTREAM_TO OrderManagement
    UPSTREAM_TO Billing
  END
END
```

### Step 4: Add Integration Patterns

```pseudocode
INTEGRATION_PATTERNS
  OrderManagement -> Inventory
    PATTERN: REST API calls
    ACL: OrderInventoryAdapter
    FREQUENCY: Real-time

  OrderManagement -> PaymentGateway
    PATTERN: Conformist (Stripe SDK)
    FREQUENCY: Per transaction

  OrderManagement <-> Shipping
    PATTERN: Event-driven collaboration
    SHARED_EVENTS: OrderShipped, ShipmentStatusChanged
END
```

---

## Visual Notation

```pseudocode
NOTATION_GUIDE:

UPSTREAM [U] ----------> [D] DOWNSTREAM
  Arrow points in direction of dependency

PARTNERSHIP [A] <-------> [B]
  Double-headed arrow

SHARED KERNEL [A] ≡≡≡≡≡≡≡≡ [B]
  Double lines indicate shared code

CONFORMIST [D] ---------> [U]
  With "CF" label on arrow

ACL [D] --[ACL]--> [U]
  Translation layer marked on arrow

OHS [Host] --[OHS]--> [Client1]
              └-------> [Client2]
  One-to-many from host

PUBLISHED LANGUAGE [Publisher] --[PL]--> [Subscriber]
  Marked with schema/format

SEPARATE WAYS [A]    [B]
  No connection drawn
```

### Example Context Map

```
┌──────────────────┐
│   E-Commerce     │
│   Storefront     │
└────────┬─────────┘
         │ CF (Conformist)
         ↓
    ┌────────────────────┐
    │  Payment Gateway   │  (External)
    │    (Stripe)        │
    └────────────────────┘

┌──────────────────┐          ┌──────────────────┐
│  Order           │ Customer │   Inventory      │
│  Management      │ Supplier │                  │
│                  │<---------|                  │
└────────┬─────────┘          └────────┬─────────┘
         │                             │
         │ Partnership                 │ OHS
         │                             ↓
         ↓                    ┌──────────────────┐
    ┌────────────────────┐    │  Product         │
    │   Shipping         │<---|  Catalog API     │
    │                    │    └──────────────────┘
    └────────────────────┘

┌──────────────────┐   Shared  ┌──────────────────┐
│  Order Mgmt      │   Kernel  │    Billing       │
│                  │≡≡≡≡≡≡≡≡≡≡≡│                  │
└──────────────────┘           └──────────────────┘

┌──────────────────┐          ┌──────────────────┐
│   Marketing      │          │  User Auth       │
│   Campaigns      │          │                  │
└──────────────────┘          └──────────────────┘
  (Separate Ways - no connection)
```

---

## Summary Table

| Relationship | Dependency | Coordination | Translation | Best For |
|--------------|------------|--------------|-------------|----------|
| **Partnership** | Mutual | High - Joint planning | None | Aligned teams, shared goals |
| **Shared Kernel** | Mutual | High - Must agree on changes | None | Small stable subset, strong consistency needs |
| **Customer-Supplier** | Downstream depends on upstream | Medium - Negotiated features | Optional | Customer has influence, formal process |
| **Conformist** | Downstream depends on upstream | None | None | No influence, upstream model acceptable |
| **Anti-Corruption Layer** | Downstream depends on upstream | None | Full isolation | Protect domain model, legacy integration |
| **Open Host Service** | Downstreams depend on upstream | Low - Published interface | Via API | Multiple consumers, stable integration |
| **Published Language** | Either direction | Medium - Schema agreement | Via schema | Interoperability, standardization |
| **Separate Ways** | None | None | N/A | Integration cost > benefit |

### Decision Matrix

```pseudocode
DECISION_TREE for choosing relationship:

IS there any integration needed?
  NO → Separate Ways
  YES ↓

IS the upstream team willing to collaborate?
  NO ↓
    IS their model acceptable?
      YES → Conformist
      NO → Anti-Corruption Layer
  YES ↓

ARE teams equal partners with mutual dependency?
  YES → Partnership
  NO ↓

CAN you share a small subset of code?
  YES (and coordination cost acceptable) → Shared Kernel
  NO ↓

ARE there multiple downstream consumers?
  YES → Open Host Service (+ Published Language)
  NO ↓

DOES downstream have negotiation power?
  YES → Customer-Supplier
  NO → Conformist or ACL
END
```

### Integration Cost vs. Model Integrity

```pseudocode
COST_BENEFIT_ANALYSIS

High Integration Cost:
  - ACL (most code)
  - Shared Kernel (coordination overhead)
  - Partnership (meeting overhead)

Low Integration Cost:
  - Conformist (minimal code)
  - OHS (if already exists)

High Model Protection:
  - ACL (complete isolation)
  - Separate Ways (no influence)

Low Model Protection:
  - Conformist (no protection)
  - Shared Kernel (partial)

RECOMMENDATION:
  IF upstream_model_quality is LOW
    AND domain_model_integrity is CRITICAL
    THEN use ACL despite cost

  IF coordination_overhead is ACCEPTABLE
    AND benefit_of_consistency is HIGH
    THEN use Shared Kernel

  IF no_negotiation_power
    AND upstream_model_acceptable
    THEN use Conformist for simplicity
END
```

---

*Based on concepts from "Domain-Driven Design" by Eric Evans.*
