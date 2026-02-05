# Composite Solution Example: Event-Driven Architecture

## Problem

Build an event-driven system with:
- Loose coupling between components
- Reliable event delivery
- Event history and replay capability
- Domain event modeling aligned with business language
- Real-time and batch processing support

## Solution Stack

### 1. Communication Pattern (gof-patterns)

**Observer Pattern** for local pub/sub

```typescript
// Basic Observer for in-process event notification
interface EventObserver {
  update(event: DomainEvent): void;
}

class EventPublisher {
  private observers: Map<string, Set<EventObserver>> = new Map();

  subscribe(eventType: string, observer: EventObserver): void {
    if (!this.observers.has(eventType)) {
      this.observers.set(eventType, new Set());
    }
    this.observers.get(eventType)!.add(observer);
  }

  unsubscribe(eventType: string, observer: EventObserver): void {
    this.observers.get(eventType)?.delete(observer);
  }

  notify(event: DomainEvent): void {
    const observers = this.observers.get(event.type) || new Set();
    observers.forEach(observer => observer.update(event));
  }
}

// Usage: Direct coupling reduced
class OrderService implements EventObserver {
  update(event: DomainEvent): void {
    if (event.type === 'payment.completed') {
      this.fulfillOrder(event.data.orderId);
    }
  }
}
```

**Mediator Pattern** for complex routing

```typescript
// Mediator for complex event routing and orchestration
interface EventMediator {
  route(event: DomainEvent): void;
}

class OrderEventMediator implements EventMediator {
  constructor(
    private inventory: InventoryService,
    private shipping: ShippingService,
    private notification: NotificationService
  ) {}

  route(event: DomainEvent): void {
    switch (event.type) {
      case 'order.placed':
        // Complex orchestration logic
        this.inventory.reserveStock(event.data.items);
        this.shipping.createShipment(event.data.orderId);
        this.notification.notifyCustomer(event.data.customerId);
        break;

      case 'order.cancelled':
        this.inventory.releaseStock(event.data.items);
        this.shipping.cancelShipment(event.data.orderId);
        break;

      case 'inventory.out_of_stock':
        // Cross-aggregate coordination
        this.notification.notifyBackorder(event.data.productId);
        break;
    }
  }
}
```

**When to use which:**
- **Observer**: Simple 1-to-many notification, minimal coordination
- **Mediator**: Complex routing logic, cross-component orchestration, prevent spaghetti observers

### 2. Domain Modeling (ddd)

**Domain Events** for business-meaningful events

```typescript
// Domain Events express business facts in ubiquitous language
abstract class DomainEvent {
  readonly eventId: string;
  readonly occurredAt: Date;
  readonly aggregateId: string;
  readonly aggregateVersion: number;

  constructor(aggregateId: string, aggregateVersion: number) {
    this.eventId = generateId();
    this.occurredAt = new Date();
    this.aggregateId = aggregateId;
    this.aggregateVersion = aggregateVersion;
  }

  abstract get type(): string;
}

// Business language, not technical language
class OrderPlaced extends DomainEvent {
  get type() { return 'order.placed'; }

  constructor(
    orderId: string,
    version: number,
    readonly customerId: string,
    readonly items: OrderItem[],
    readonly totalAmount: Money
  ) {
    super(orderId, version);
  }
}

class PaymentFailed extends DomainEvent {
  get type() { return 'payment.failed'; }

  constructor(
    orderId: string,
    version: number,
    readonly reason: string,
    readonly retryable: boolean
  ) {
    super(orderId, version);
  }
}
```

**Aggregate as event boundary**

```typescript
// Aggregates emit events at consistency boundaries
class Order {
  private uncommittedEvents: DomainEvent[] = [];
  private version: number = 0;

  place(customerId: string, items: OrderItem[]): void {
    // Business invariant checks
    if (items.length === 0) {
      throw new Error('Order must have items');
    }

    // State change
    this.status = 'placed';

    // Emit domain event
    this.addEvent(new OrderPlaced(
      this.id,
      ++this.version,
      customerId,
      items,
      this.calculateTotal(items)
    ));
  }

  cancel(reason: string): void {
    if (this.status === 'shipped') {
      throw new Error('Cannot cancel shipped order');
    }

    this.status = 'cancelled';
    this.addEvent(new OrderCancelled(this.id, ++this.version, reason));
  }

  private addEvent(event: DomainEvent): void {
    this.uncommittedEvents.push(event);
  }

  getUncommittedEvents(): DomainEvent[] {
    return [...this.uncommittedEvents];
  }

  clearUncommittedEvents(): void {
    this.uncommittedEvents = [];
  }
}
```

**Event Storming** for discovery

```
1. Orange stickies: Domain Events (OrderPlaced, PaymentFailed)
2. Blue stickies: Commands (PlaceOrder, CancelOrder)
3. Yellow stickies: Aggregates (Order, Payment, Inventory)
4. Pink stickies: External systems (PaymentGateway, ShippingAPI)
5. Purple stickies: Policies (When PaymentFailed → NotifyCustomer)

Timeline:
[Customer] PlaceOrder → [OrderPlaced] → [ReserveInventory] → [InventoryReserved]
                                      ↓
                                   [ProcessPayment] → [PaymentCompleted] → [ShipOrder]
                                                   ↓
                                                [PaymentFailed] → [NotifyCustomer]
```

### 3. Processing (ddia)

**Event Sourcing** for event history

```typescript
// Event Store: append-only log of events
interface EventStore {
  append(streamId: string, events: DomainEvent[]): Promise<void>;
  read(streamId: string, fromVersion?: number): Promise<DomainEvent[]>;
  readAll(fromPosition?: number): Promise<DomainEvent[]>;
}

class EventSourcedRepository<T> {
  constructor(
    private eventStore: EventStore,
    private factory: (id: string) => T
  ) {}

  async load(aggregateId: string): Promise<T> {
    const events = await this.eventStore.read(aggregateId);

    const aggregate = this.factory(aggregateId);
    this.rehydrate(aggregate, events);

    return aggregate;
  }

  async save(aggregate: Order): Promise<void> {
    const events = aggregate.getUncommittedEvents();

    if (events.length > 0) {
      await this.eventStore.append(aggregate.id, events);
      aggregate.clearUncommittedEvents();

      // Publish for projections and side effects
      events.forEach(e => this.eventBus.publish(e));
    }
  }

  private rehydrate(aggregate: any, events: DomainEvent[]): void {
    events.forEach(event => aggregate.apply(event));
  }
}

// Aggregate reconstructed from events
class Order {
  apply(event: DomainEvent): void {
    switch (event.type) {
      case 'order.placed':
        this.applyOrderPlaced(event as OrderPlaced);
        break;
      case 'order.cancelled':
        this.applyOrderCancelled(event as OrderCancelled);
        break;
    }
  }

  private applyOrderPlaced(event: OrderPlaced): void {
    this.id = event.aggregateId;
    this.customerId = event.customerId;
    this.items = event.items;
    this.status = 'placed';
    this.version = event.aggregateVersion;
  }

  private applyOrderCancelled(event: OrderCancelled): void {
    this.status = 'cancelled';
    this.version = event.aggregateVersion;
  }
}
```

**Stream Processing** for real-time handling

```typescript
// Stream processor for real-time reactions
class EventStream {
  constructor(private eventStore: EventStore) {}

  // Subscribe to event stream
  async subscribe(
    fromPosition: number,
    handler: (event: DomainEvent) => Promise<void>
  ): Promise<void> {
    let position = fromPosition;

    while (true) {
      const events = await this.eventStore.readAll(position);

      for (const event of events) {
        await handler(event);
        position = event.position;
      }

      await this.waitForNewEvents();
    }
  }
}

// Real-time projection
class OrderSummaryProjection {
  constructor(private db: Database) {}

  async handle(event: DomainEvent): Promise<void> {
    switch (event.type) {
      case 'order.placed':
        await this.db.execute(
          'INSERT INTO order_summary (id, customer_id, status, total) VALUES (?, ?, ?, ?)',
          [event.aggregateId, event.customerId, 'placed', event.totalAmount]
        );
        break;

      case 'order.cancelled':
        await this.db.execute(
          'UPDATE order_summary SET status = ? WHERE id = ?',
          ['cancelled', event.aggregateId]
        );
        break;
    }
  }
}

// Event replay for new projections
async function rebuildProjection(
  eventStore: EventStore,
  projection: OrderSummaryProjection
): Promise<void> {
  const allEvents = await eventStore.readAll(0);

  for (const event of allEvents) {
    await projection.handle(event);
  }
}
```

## Complete Implementation Sketch

```typescript
// 1. Domain Event (DDD) + Observer Pattern (GoF)
class EventBus {
  private observers: Map<string, Set<EventObserver>> = new Map();
  private mediators: EventMediator[] = [];

  subscribe(eventType: string, observer: EventObserver): void {
    if (!this.observers.has(eventType)) {
      this.observers.set(eventType, new Set());
    }
    this.observers.get(eventType)!.add(observer);
  }

  registerMediator(mediator: EventMediator): void {
    this.mediators.push(mediator);
  }

  publish(event: DomainEvent): void {
    // Notify direct observers
    const observers = this.observers.get(event.type) || new Set();
    observers.forEach(o => o.update(event));

    // Route through mediators for complex logic
    this.mediators.forEach(m => m.route(event));
  }
}

// 2. Event Store (DDIA)
class PostgresEventStore implements EventStore {
  async append(streamId: string, events: DomainEvent[]): Promise<void> {
    await this.db.transaction(async (tx) => {
      for (const event of events) {
        // Optimistic concurrency check
        const current = await tx.query(
          'SELECT MAX(version) as version FROM events WHERE stream_id = ?',
          [streamId]
        );

        if (current.version >= event.aggregateVersion) {
          throw new ConcurrencyError();
        }

        // Append event
        await tx.execute(
          `INSERT INTO events (
            event_id, stream_id, event_type, event_data,
            version, occurred_at
          ) VALUES (?, ?, ?, ?, ?, ?)`,
          [
            event.eventId,
            streamId,
            event.type,
            JSON.stringify(event),
            event.aggregateVersion,
            event.occurredAt
          ]
        );
      }
    });
  }

  async read(streamId: string, fromVersion = 0): Promise<DomainEvent[]> {
    const rows = await this.db.query(
      'SELECT event_data FROM events WHERE stream_id = ? AND version >= ? ORDER BY version',
      [streamId, fromVersion]
    );

    return rows.map(r => this.deserialize(r.event_data));
  }

  async readAll(fromPosition = 0): Promise<DomainEvent[]> {
    const rows = await this.db.query(
      'SELECT event_data FROM events WHERE position > ? ORDER BY position',
      [fromPosition]
    );

    return rows.map(r => this.deserialize(r.event_data));
  }
}

// 3. Complete system assembly
class EventDrivenSystem {
  private eventBus: EventBus;
  private eventStore: EventStore;
  private repository: EventSourcedRepository<Order>;

  constructor() {
    this.eventStore = new PostgresEventStore(db);
    this.eventBus = new EventBus();
    this.repository = new EventSourcedRepository(
      this.eventStore,
      (id) => new Order(id)
    );

    this.setupProjections();
    this.setupReactions();
  }

  private setupProjections(): void {
    const orderSummary = new OrderSummaryProjection(readDb);
    const stream = new EventStream(this.eventStore);

    // Real-time projection updates
    stream.subscribe(0, (event) => orderSummary.handle(event));
  }

  private setupReactions(): void {
    // Register mediator for complex orchestration
    const orderMediator = new OrderEventMediator(
      new InventoryService(),
      new ShippingService(),
      new NotificationService()
    );
    this.eventBus.registerMediator(orderMediator);

    // Register simple observers
    this.eventBus.subscribe('order.placed', {
      update: (event) => analytics.track(event)
    });
  }

  async placeOrder(
    orderId: string,
    customerId: string,
    items: OrderItem[]
  ): Promise<void> {
    // Load aggregate
    const order = await this.repository.load(orderId);

    // Execute command
    order.place(customerId, items);

    // Save (appends events + publishes)
    await this.repository.save(order);
  }
}
```

## Usage Example

```typescript
// Initialize system
const system = new EventDrivenSystem();

// Handle command
await system.placeOrder('order-123', 'customer-456', [
  { productId: 'prod-1', quantity: 2 }
]);

// What happens:
// 1. Order aggregate emits OrderPlaced event
// 2. Event appended to event store (DDIA)
// 3. Event published on bus (GoF Observer)
// 4. Mediator routes to inventory, shipping, notification (GoF Mediator)
// 5. Projections update read models (DDIA Stream Processing)
// 6. Analytics observers track event

// Event replay for debugging
const events = await eventStore.read('order-123');
events.forEach(e => console.log(e.type, e.occurredAt));

// Rebuild projection from history
await rebuildProjection(eventStore, new OrderSummaryProjection(db));
```

## Scaling Considerations

### Event Ordering

**Problem**: Distributed consumers may process events out of order

**Solutions**:
```typescript
// Per-stream ordering (partition by aggregate ID)
class PartitionedEventStream {
  async publish(event: DomainEvent): Promise<void> {
    const partition = hashCode(event.aggregateId) % NUM_PARTITIONS;
    await this.queues[partition].enqueue(event);
  }
}

// Global ordering (single partition, lower throughput)
class OrderedEventStream {
  private sequence: number = 0;

  async publish(event: DomainEvent): Promise<void> {
    event.sequence = ++this.sequence;
    await this.queue.enqueue(event);
  }
}
```

**Trade-off**: Per-stream ordering scales better but allows cross-stream races

### Exactly-Once Delivery

**Problem**: Network failures can cause duplicate event processing

**Solutions**:
```typescript
// Idempotency key in event
class DomainEvent {
  readonly eventId: string; // UUID, globally unique
}

// Idempotent projection
class IdempotentProjection {
  async handle(event: DomainEvent): Promise<void> {
    // Check if already processed
    const processed = await this.db.query(
      'SELECT 1 FROM processed_events WHERE event_id = ?',
      [event.eventId]
    );

    if (processed.length > 0) {
      return; // Already handled
    }

    // Process in transaction with deduplication
    await this.db.transaction(async (tx) => {
      await this.updateProjection(tx, event);
      await tx.execute(
        'INSERT INTO processed_events (event_id, processed_at) VALUES (?, ?)',
        [event.eventId, new Date()]
      );
    });
  }
}

// At-least-once with idempotency = exactly-once semantics
```

### Event Schema Evolution

**Problem**: Events are immutable but domain changes over time

**Solutions**:
```typescript
// Version events explicitly
class OrderPlacedV1 extends DomainEvent {
  get type() { return 'order.placed'; }
  get version() { return 1; }
}

class OrderPlacedV2 extends DomainEvent {
  get type() { return 'order.placed'; }
  get version() { return 2; }
  readonly shippingAddress: Address; // New field
}

// Upcasting during deserialization
class EventDeserializer {
  deserialize(data: any): DomainEvent {
    const version = data.version || 1;

    switch (data.type) {
      case 'order.placed':
        if (version === 1) {
          return this.upcastOrderPlacedV1toV2(data);
        }
        return new OrderPlacedV2(data);
    }
  }

  private upcastOrderPlacedV1toV2(data: any): OrderPlacedV2 {
    return new OrderPlacedV2({
      ...data,
      shippingAddress: this.deriveAddress(data.customerId)
    });
  }
}
```

**Alternative**: Weak schema (JSON blobs) with runtime validation

## Trade-offs

| Decision | Choice | Trade-off |
|----------|--------|-----------|
| **Event granularity** | One event per state change | More events = more history but easier replay. Consider snapshotting for long streams |
| **Event storage** | Append-only log | Can't delete events. Need GDPR-compliant encryption/redaction strategy |
| **Consistency** | Eventual (async projections) | Read model lag. Use command-side read for immediate consistency needs |
| **Ordering guarantees** | Per-aggregate ordering | Scales well but cross-aggregate ordering requires coordination (slower) |
| **Delivery semantics** | At-least-once + idempotency | Simpler than exactly-once but requires idempotent handlers |
| **Event versioning** | Upcasting on read | No event migration but complex deserialization. Alternative: weak schema |

## Debugging Challenges

### Lost Event
```bash
# Check event was written
SELECT * FROM events WHERE event_id = ?

# Check projection consumed it
SELECT * FROM processed_events WHERE event_id = ?

# Check projection state
SELECT * FROM order_summary WHERE id = ?
```

### Out-of-Order Processing
```bash
# Check event sequence
SELECT event_id, version, occurred_at FROM events
WHERE stream_id = ? ORDER BY version

# Check projection version
SELECT last_processed_version FROM projection_state
WHERE projection = 'order_summary'
```

### Performance Issues
```bash
# Check event store query performance
EXPLAIN SELECT * FROM events WHERE stream_id = ? AND version >= ?

# Consider snapshots for long streams
CREATE TABLE snapshots (
  stream_id VARCHAR,
  version INT,
  snapshot_data JSON,
  PRIMARY KEY (stream_id, version)
)
```

## When to Use This Stack

✅ **Good fit:**
- Audit requirements (need full history)
- Complex business processes with many steps
- Temporal queries ("what was the state at time T?")
- Event-driven integrations with external systems
- CQRS (separate read/write models)
- Domain with clear business events

❌ **Poor fit:**
- Simple CRUD applications
- Strong consistency required across aggregates
- Team unfamiliar with event sourcing complexity
- Queries primarily on current state (event replay overhead)
- High event volume with no batching (storage cost)

## Migration Path

**Start simple, add complexity as needed:**

1. **Phase 1**: Observer pattern for in-process events
2. **Phase 2**: Add event log for audit trail (keep primary state in DB)
3. **Phase 3**: Add projections for specific read models
4. **Phase 4**: Full event sourcing for critical aggregates
5. **Phase 5**: Distributed event streaming (Kafka, etc.)

## References

- **GoF**: Observer (293), Mediator (273)
- **DDD**: Chapter 8 (Domain Events), Chapter 10 (Aggregates)
- **DDIA**: Chapter 11 (Stream Processing), Chapter 12 (Event Sourcing)
- **Patterns**: Event Sourcing (Fowler), CQRS (Young)
