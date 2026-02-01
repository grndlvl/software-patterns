# Domain Events

## Definition

A **Domain Event** represents something that happened in the domain that is significant to domain experts. It captures a fact about the past—something that has already occurred and cannot be changed. Domain events enable communication within and across bounded contexts through asynchronous, event-driven patterns.

Key principle: **Use past-tense naming** to reflect that the event describes something already completed.

---

## Characteristics

Domain Events share the following defining characteristics:

| Characteristic | Description |
|---|---|
| **Immutability** | Once created, a domain event cannot be modified. This ensures the historical record remains accurate. |
| **Timestamped** | Contains a creation timestamp indicating when the event occurred. |
| **Unique Identity** | Has a unique identifier (event ID) to track individual event occurrences. |
| **Relevant Data** | Carries all information needed by subscribers to react to the event. |
| **Publish Date** | May differ from event occurrence date in eventual consistency scenarios. |
| **Aggregate Root Sourced** | Typically raised by an aggregate root in response to a domain action. |
| **Serializable** | Can be serialized/deserialized for storage and transmission. |

---

## Event Naming Conventions

Domain events use **past-tense verbs** to reflect completed actions:

### Good Event Names
- `OrderPlaced` — captures that an order was created
- `PaymentReceived` — indicates payment has been collected
- `InventoryReserved` — shows items have been reserved
- `CustomerRegistered` — reflects account creation
- `ShipmentDispatched` — indicates goods were sent

### Poor Event Names
- `PlaceOrder` — implies future action (use commands instead)
- `Payment` — too vague, unclear what happened
- `UpdateInventory` — not specific about what changed
- `Order` — noun only, unclear intent

### Namespace Convention
```
[BoundedContext].[Aggregate].[EventName]

Examples:
- order.order.OrderPlaced
- payment.payment.PaymentReceived
- inventory.stock.InventoryReserved
```

---

## Pseudocode: Core Structures

### Event Class Structure

```pseudocode
abstract class DomainEvent {
  // Immutable identity
  eventId: UUID
  eventType: String
  aggregateId: UUID
  aggregateType: String
  
  // Temporal tracking
  occurredAt: DateTime
  publishedAt: DateTime (nullable, until published)
  
  // Versioning for evolution
  eventVersion: Integer
  
  constructor(aggregateId: UUID, aggregateType: String) {
    this.eventId = UUID.generate()
    this.aggregateId = aggregateId
    this.aggregateType = aggregateType
    this.occurredAt = DateTime.now()
    this.publishedAt = null
    this.eventVersion = 1
  }
  
  // Mark as published
  markPublished(at: DateTime = DateTime.now()): void {
    this.publishedAt = at
  }
  
  // Serialize for storage/transmission
  toJSON(): Object {
    return {
      eventId: this.eventId,
      eventType: this.eventType,
      aggregateId: this.aggregateId,
      aggregateType: this.aggregateType,
      occurredAt: this.occurredAt,
      publishedAt: this.publishedAt,
      eventVersion: this.eventVersion,
      data: this.eventSpecificData()
    }
  }
  
  // Subclasses implement specific data
  abstract eventSpecificData(): Object
}
```

### Concrete Event Example: OrderPlaced

```pseudocode
class OrderPlaced extends DomainEvent {
  orderId: UUID
  customerId: UUID
  orderItems: List<OrderItem>
  totalAmount: Money
  orderDate: DateTime
  
  constructor(
    orderId: UUID,
    customerId: UUID,
    orderItems: List<OrderItem>,
    totalAmount: Money
  ) {
    super(orderId, "Order")
    this.orderId = orderId
    this.customerId = customerId
    this.orderItems = orderItems
    this.totalAmount = totalAmount
    this.orderDate = DateTime.now()
  }
  
  eventSpecificData(): Object {
    return {
      orderId: this.orderId,
      customerId: this.customerId,
      items: this.orderItems.map(item => ({
        productId: item.productId,
        quantity: item.quantity,
        unitPrice: item.unitPrice
      })),
      totalAmount: this.totalAmount.value,
      totalCurrency: this.totalAmount.currency,
      orderDate: this.orderDate
    }
  }
}
```

### Event Publisher Interface

```pseudocode
interface EventPublisher {
  publish(event: DomainEvent): void
  publishAll(events: List<DomainEvent>): void
}

class DomainEventPublisher implements EventPublisher {
  subscribers: List<EventSubscriber>
  eventStore: EventStore
  
  constructor(eventStore: EventStore) {
    this.subscribers = []
    this.eventStore = eventStore
  }
  
  subscribe(subscriber: EventSubscriber): void {
    this.subscribers.add(subscriber)
  }
  
  publish(event: DomainEvent): void {
    // 1. Save to event store (immutable log)
    this.eventStore.append(event)
    
    // 2. Mark as published
    event.markPublished()
    
    // 3. Notify all subscribers
    for each subscriber in this.subscribers {
      if subscriber.handles(event.eventType) {
        try {
          subscriber.handle(event)
        } catch error {
          // Log error, optionally queue for retry
          logError("Event handler failed", error)
        }
      }
    }
  }
  
  publishAll(events: List<DomainEvent>): void {
    for each event in events {
      this.publish(event)
    }
  }
}
```

### Event Subscriber Interface

```pseudocode
interface EventSubscriber {
  handles(eventType: String): Boolean
  handle(event: DomainEvent): void
}

// Concrete subscriber example: Email notification on order placement
class SendOrderConfirmationEmail implements EventSubscriber {
  emailService: EmailService
  customerRepository: CustomerRepository
  
  constructor(emailService: EmailService, customerRepository: CustomerRepository) {
    this.emailService = emailService
    this.customerRepository = customerRepository
  }
  
  handles(eventType: String): Boolean {
    return eventType == "OrderPlaced"
  }
  
  handle(event: DomainEvent): void {
    if not event instanceof OrderPlaced {
      return
    }
    
    orderPlaced = event as OrderPlaced
    customer = this.customerRepository.findById(orderPlaced.customerId)
    
    if customer != null {
      this.emailService.sendConfirmation(
        customer.email,
        orderPlaced.orderId,
        orderPlaced.totalAmount
      )
    }
  }
}
```

### Publishing Events from Aggregate Roots

```pseudocode
class Order extends AggregateRoot {
  orderId: UUID
  customerId: UUID
  items: List<LineItem>
  status: OrderStatus
  pendingEvents: List<DomainEvent>
  
  constructor(orderId: UUID, customerId: UUID) {
    super(orderId)
    this.customerId = customerId
    this.items = []
    this.status = OrderStatus.PENDING
    this.pendingEvents = []
  }
  
  // Business operation that raises an event
  placeOrder(items: List<LineItem>): void {
    // Validate business rules
    if items.isEmpty() {
      throw new InvalidOrderException("Order must have at least one item")
    }
    
    if this.status != OrderStatus.PENDING {
      throw new InvalidOrderException("Cannot place order in " + this.status)
    }
    
    // Update state
    this.items = items
    this.status = OrderStatus.PLACED
    
    // Calculate total (simplified)
    totalAmount = Money.sum(
      items.map(item => item.unitPrice.multiply(item.quantity))
    )
    
    // Raise domain event
    event = new OrderPlaced(this.id, this.customerId, items, totalAmount)
    this.pendingEvents.add(event)
  }
  
  // Retrieve unpublished events for persistence layer to publish
  getPendingEvents(): List<DomainEvent> {
    return this.pendingEvents.copy()
  }
  
  // Clear pending events after successful publication
  clearPendingEvents(): void {
    this.pendingEvents.clear()
  }
}
```

### Repository Integration

```pseudocode
class OrderRepository {
  database: Database
  eventPublisher: EventPublisher
  
  // Save aggregate and publish its events
  save(order: Order): void {
    // 1. Persist aggregate state
    this.database.insert("orders", {
      id: order.id,
      customerId: order.customerId,
      status: order.status,
      createdAt: DateTime.now()
    })
    
    // 2. Publish all pending events
    pendingEvents = order.getPendingEvents()
    for each event in pendingEvents {
      this.eventPublisher.publish(event)
    }
    
    // 3. Clear pending events
    order.clearPendingEvents()
  }
}
```

---

## Use Cases and Patterns

### 1. Eventual Consistency Across Bounded Contexts

```pseudocode
// OrderBoundedContext
class Order extends AggregateRoot {
  placeOrder(items: List<LineItem>): void {
    // ... validate and update state ...
    this.pendingEvents.add(new OrderPlaced(...))
  }
}

// InventoryBoundedContext listens to OrderPlaced
class ReserveInventoryOnOrderPlaced implements EventSubscriber {
  inventoryService: InventoryService
  
  handles(eventType: String): Boolean {
    return eventType == "OrderPlaced"
  }
  
  handle(event: DomainEvent): void {
    orderPlaced = event as OrderPlaced
    for each item in orderPlaced.orderItems {
      this.inventoryService.reserve(item.productId, item.quantity)
    }
  }
}

// ShippingBoundedContext listens independently
class RegisterShipmentOnOrderPlaced implements EventSubscriber {
  shippingService: ShippingService
  
  handles(eventType: String): Boolean {
    return eventType == "OrderPlaced"
  }
  
  handle(event: DomainEvent): void {
    orderPlaced = event as OrderPlaced
    this.shippingService.registerShipment(orderPlaced.orderId)
  }
}
```

Benefits:
- Bounded contexts remain loosely coupled
- Changes in one context don't require changes in others
- Multiple subscribers can react independently
- Enables asynchronous processing

### 2. Audit Trail and Event Sourcing

```pseudocode
// All events stored in immutable event log
class EventStore {
  events: List<DomainEvent>
  
  // Append only - never update or delete
  append(event: DomainEvent): void {
    this.events.add(event)
    // Persist to durable storage
    this.persistToDatabase(event)
  }
  
  // Retrieve full history for an aggregate
  getEventsFor(aggregateId: UUID): List<DomainEvent> {
    return this.events.filter(e => e.aggregateId == aggregateId)
  }
  
  // Replay events to rebuild aggregate state
  reconstructAggregate(aggregateId: UUID): AggregateRoot {
    events = this.getEventsFor(aggregateId)
    aggregate = new AggregateRoot(aggregateId)
    
    for each event in events {
      aggregate.applyEvent(event)
    }
    
    return aggregate
  }
}

// Audit trail - all changes are captured
class QueryOrderHistory {
  eventStore: EventStore
  
  getAllChanges(orderId: UUID): List<Object> {
    events = this.eventStore.getEventsFor(orderId)
    return events.map(event => ({
      what: event.eventType,
      when: event.occurredAt,
      details: event.toJSON()
    }))
  }
}
```

### 3. Integration with External Systems

```pseudocode
// Payment system receives OrderPlaced and initiates charge
class InitiatePaymentOnOrderPlaced implements EventSubscriber {
  paymentGateway: PaymentGateway
  
  handles(eventType: String): Boolean {
    return eventType == "OrderPlaced"
  }
  
  handle(event: DomainEvent): void {
    orderPlaced = event as OrderPlaced
    
    // Call external system asynchronously
    this.paymentGateway.chargeAsync(
      orderId: orderPlaced.orderId,
      amount: orderPlaced.totalAmount,
      onSuccess: PaymentReceivedCallback(),
      onFailure: PaymentFailedCallback()
    )
  }
}

// When external system completes, raise new event
class PaymentReceivedCallback {
  orderService: OrderService
  
  onSuccess(paymentResult: Object): void {
    orderId = paymentResult.orderId
    amount = paymentResult.amount
    
    // Raise domain event
    event = new PaymentReceived(orderId, amount, paymentResult.transactionId)
    this.orderService.handlePaymentReceived(event)
  }
}
```

---

## Event vs Command Comparison

| Aspect | Event | Command |
|---|---|---|
| **Tense** | Past (OrderPlaced) | Imperative (PlaceOrder) |
| **Intent** | Notification of what happened | Request for something to happen |
| **Authority** | Fact (non-negotiable) | Request (may be rejected) |
| **Response** | No explicit response expected | Handler must validate and accept/reject |
| **Async Nature** | Fire-and-forget | Can be synchronous or async |
| **Idempotency** | Multiple identical events mean it happened multiple times | Must be idempotent; retry-safe |
| **Storage** | Often persisted in event store | Usually not persisted |
| **Causality** | Result of a command or policy | Initiated by external actor/system |
| **Ownership** | Published by domain logic | Accepted by application/command handler |
| **Example** | OrderPlaced (published when order is created) | PlaceOrder (command to create order) |

**Relationship:**
```
External Actor → Command → Domain Logic → Event → Other Domains
  User issues     Handler    Validates and     Notifies    React and
  PlaceOrder      processes  publishes          interested   update
  command         command    OrderPlaced       systems      state
```

---

## Summary Table

| Concept | Purpose | Key Features |
|---|---|---|
| **Domain Event** | Represent significant facts that occurred in the domain | Immutable, timestamped, past-tense named |
| **Event Publisher** | Distribute events to interested subscribers | Persists to event store, notifies subscribers |
| **Event Subscriber** | React to domain events | Filters by event type, handles asynchronously |
| **Event Store** | Maintain immutable audit trail | Append-only, enables event sourcing |
| **Aggregate Root** | Raise events when state changes | Maintains pending events until published |
| **Repository** | Persist aggregate and publish events | Coordinates state and event persistence |
| **Event Sourcing** | Rebuild state by replaying events | Full history of all changes captured |
| **Eventual Consistency** | Coordinate across bounded contexts | Asynchronous event-based updates |

---

## Key Principles

1. **Events are Facts** — Once published, they're part of the historical record and cannot be undone
2. **Async by Default** — Events enable asynchronous communication between bounded contexts
3. **Subscribers are Loosely Coupled** — They don't know about each other; they only care about event types
4. **Events Drive Discovery** — The set of events published reveals the domain's language and behavior
5. **Archive Events** — Even published events should be stored for audit and debugging
6. **Avoid Event Chains** — If event A always triggers event B which triggers event C, consider using policies instead

---

*Based on concepts from "Domain-Driven Design" by Eric Evans.*
