# State

## Intent

Allow an object to alter its behavior when its internal state changes. The object will appear to change its class.

## Also Known As

- Objects for States
- State Machine
- Finite State Machine (FSM)

## Motivation

Consider a document in a content management system. A document can be in various states: Draft, Pending Review, Under Review, Approved, Published, or Archived. The operations available depend on the current state. A Draft can be submitted for review but not published. A Published document can be archived but not submitted for review. And the same action (like "approve") behaves differently depending on who performs it and the current state.

The naive approach uses conditionals everywhere: `if (state == DRAFT) ... else if (state == PENDING_REVIEW) ...`. This scatters state-specific logic throughout the code, making it hard to understand what a document can do in any given state. Adding a new state means hunting through every method to add another condition.

The State pattern solves this by encapsulating state-specific behavior in separate state classes. Each state class implements the full interface of operations, but each method behaves appropriately for that state. The document holds a reference to its current state object and delegates operations to it. When the state changes, the document swaps its state object, and subsequent operations automatically use the new state's behavior.

This makes the state machine explicit. Each state class clearly documents what operations are valid in that state. Adding a new state is straightforward—create a new state class. The document's code remains clean, simply delegating to its current state.

## Applicability

Use the State pattern when:

- An object's behavior depends on its state, and it must change behavior at runtime depending on that state.
- Operations have large, multipart conditional statements that depend on the object's state. The State pattern puts each branch in a separate class.
- You have a finite state machine with clearly defined states and transitions.
- State transitions are complex and you want to make them explicit and manageable.
- You want to avoid large switch/case statements or if/else chains based on state.

Common applications include:
- Document workflows (draft, review, published)
- Order processing (pending, paid, shipped, delivered)
- Connection handling (connecting, connected, disconnecting)
- Game entity states (idle, walking, running, jumping, attacking)
- Media players (stopped, playing, paused)
- UI component states (enabled, disabled, focused, loading)
- Protocol implementations (handshaking, authenticated, transferring)

## Structure

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│    ┌───────────────────────────────────────────────────────────────────┐   │
│    │                         Context                                   │   │
│    ├───────────────────────────────────────────────────────────────────┤   │
│    │ - state: State                                                    │   │
│    ├───────────────────────────────────────────────────────────────────┤   │
│    │ + request(): void           // Delegates to state.handle(this)   │   │
│    │ + setState(state: State): void                                    │   │
│    │ + getState(): State                                               │   │
│    └───────────────────────────────────────────────────────────────────┘   │
│                │                                                            │
│                │ delegates to                                               │
│                ▼                                                            │
│    ┌───────────────────────────────────────────────────────────────────┐   │
│    │                    <<interface>> State                            │   │
│    ├───────────────────────────────────────────────────────────────────┤   │
│    │ + handle(context: Context): void                                  │   │
│    │ + canTransitionTo(state: State): boolean                          │   │
│    │ + getAvailableActions(): list<string>                             │   │
│    └───────────────────────────────────────────────────────────────────┘   │
│                                    △                                        │
│                                    │                                        │
│              ┌─────────────────────┼─────────────────────┐                 │
│              │                     │                     │                 │
│    ┌─────────┴─────────┐ ┌────────┴────────┐ ┌─────────┴─────────┐        │
│    │  ConcreteStateA   │ │ ConcreteStateB  │ │  ConcreteStateC   │        │
│    ├───────────────────┤ ├─────────────────┤ ├───────────────────┤        │
│    │                   │ │                 │ │                   │        │
│    ├───────────────────┤ ├─────────────────┤ ├───────────────────┤        │
│    │ + handle(context) │ │ + handle(ctx)   │ │ + handle(context) │        │
│    │   // State A      │ │   // State B    │ │   // State C      │        │
│    │   // behavior     │ │   // behavior   │ │   // behavior     │        │
│    └───────────────────┘ └─────────────────┘ └───────────────────┘        │
│                                                                             │
│   State Transition Diagram:                                                 │
│                                                                             │
│         ┌─────────┐   event1    ┌─────────┐   event2    ┌─────────┐       │
│         │ State A │────────────>│ State B │────────────>│ State C │       │
│         └─────────┘             └─────────┘             └─────────┘       │
│              │                       │                       │             │
│              │        event3         │                       │             │
│              └───────────────────────┘                       │             │
│                                                              │             │
│              ┌───────────────────────────────────────────────┘             │
│              │  event4                                                     │
│              ▼                                                             │
│         ┌─────────┐                                                        │
│         │ State A │  (cycle back)                                          │
│         └─────────┘                                                        │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Participants

- **Context**: Maintains an instance of a ConcreteState subclass that defines the current state. Defines the interface of interest to clients. Delegates state-specific requests to the current state object.

- **State**: Defines an interface for encapsulating the behavior associated with a particular state of the Context.

- **ConcreteState**: Each subclass implements behavior associated with a state of the Context. Contains the logic for that specific state and can trigger state transitions.

## Collaborations

1. Context delegates state-specific requests to the current ConcreteState object.

2. A context may pass itself as an argument to the state object handling the request. This lets the state object access the context's data and trigger state transitions.

3. Either the Context or the ConcreteState subclasses can decide which state succeeds another and under what circumstances (transition logic).

4. Clients interact with the context, not directly with state objects. The context provides a uniform interface regardless of the current state.

## Consequences

### Benefits

1. **Localizes state-specific behavior**: All behavior for a particular state is in one class. This makes it easy to add new states by defining new classes.

2. **Makes state transitions explicit**: Using separate objects for different states makes transitions more explicit. It's clear when and why the object changes state.

3. **State objects can be shared**: If state objects have no instance variables (they store all data in the context), they can be shared among contexts as flyweights.

4. **Eliminates conditional statements**: The State pattern replaces bulky conditional code with polymorphism.

5. **Open/Closed Principle**: You can introduce new states without changing existing state classes or the context.

6. **Single Responsibility Principle**: Organize code related to particular states into separate classes.

### Liabilities

1. **Increased number of classes**: Each state requires its own class, which can lead to many small classes.

2. **Transitions can be scattered**: If states determine their own transitions, the logic is distributed across state classes. This can make the overall state machine harder to understand.

3. **May be overkill for simple state machines**: For objects with few states and simple transitions, the pattern adds unnecessary complexity.

## Implementation

### Implementation Considerations

1. **Who defines state transitions?**:
   - States: Each state knows its possible successors. More flexible but scatters transition logic.
   - Context: Centralizes transitions but makes states less self-contained.
   - Transition table: External configuration, most flexible but adds complexity.

2. **Creating and destroying state objects**:
   - Create on demand: Simple but may have performance cost.
   - Create once, reuse: More efficient if states are stateless.
   - Flyweight: Share state instances across contexts.

3. **Using language features**: Many languages have enum-based state machines or pattern matching that can simplify implementation.

### Pseudocode: State Infrastructure

```
// State interface
interface DocumentState {
    method submit(document: Document): void
    method approve(document: Document): void
    method reject(document: Document): void
    method publish(document: Document): void
    method archive(document: Document): void
    method edit(document: Document, content: string): void

    method getName(): string
    method getAvailableActions(): list<string>
    method canTransitionTo(targetState: string): boolean
}

// Base state with default implementations
abstract class BaseDocumentState implements DocumentState {
    method submit(document: Document): void {
        throw new IllegalStateException("Cannot submit in " + this.getName() + " state")
    }

    method approve(document: Document): void {
        throw new IllegalStateException("Cannot approve in " + this.getName() + " state")
    }

    method reject(document: Document): void {
        throw new IllegalStateException("Cannot reject in " + this.getName() + " state")
    }

    method publish(document: Document): void {
        throw new IllegalStateException("Cannot publish in " + this.getName() + " state")
    }

    method archive(document: Document): void {
        throw new IllegalStateException("Cannot archive in " + this.getName() + " state")
    }

    method edit(document: Document, content: string): void {
        throw new IllegalStateException("Cannot edit in " + this.getName() + " state")
    }

    method canTransitionTo(targetState: string): boolean {
        return targetState in this.getAvailableActions()
    }
}
```

### Pseudocode: Concrete States

```
// Draft state
class DraftState extends BaseDocumentState {
    method getName(): string {
        return "Draft"
    }

    method getAvailableActions(): list<string> {
        return ["edit", "submit", "archive"]
    }

    method edit(document: Document, content: string): void {
        document.setContent(content)
        document.setLastModified(now())
        log("Document edited in draft state")
    }

    method submit(document: Document): void {
        if document.getContent().isEmpty() {
            throw new ValidationException("Cannot submit empty document")
        }
        document.setState(new PendingReviewState())
        document.setSubmittedAt(now())
        document.notifyReviewers()
        log("Document submitted for review")
    }

    method archive(document: Document): void {
        document.setState(new ArchivedState())
        document.setArchivedAt(now())
        log("Draft archived")
    }
}

// Pending Review state
class PendingReviewState extends BaseDocumentState {
    method getName(): string {
        return "Pending Review"
    }

    method getAvailableActions(): list<string> {
        return ["approve", "reject"]
    }

    method approve(document: Document): void {
        document.setState(new ApprovedState())
        document.setApprovedAt(now())
        document.setApprovedBy(getCurrentUser())
        document.notifyAuthor("approved")
        log("Document approved")
    }

    method reject(document: Document): void {
        document.setState(new DraftState())
        document.setRejectedAt(now())
        document.setRejectedBy(getCurrentUser())
        document.notifyAuthor("rejected")
        log("Document rejected, returned to draft")
    }
}

// Approved state
class ApprovedState extends BaseDocumentState {
    method getName(): string {
        return "Approved"
    }

    method getAvailableActions(): list<string> {
        return ["publish", "reject"]
    }

    method publish(document: Document): void {
        document.setState(new PublishedState())
        document.setPublishedAt(now())
        document.generatePublicUrl()
        document.notifySubscribers()
        log("Document published")
    }

    method reject(document: Document): void {
        document.setState(new DraftState())
        document.clearApproval()
        log("Approved document rejected, returned to draft")
    }
}

// Published state
class PublishedState extends BaseDocumentState {
    method getName(): string {
        return "Published"
    }

    method getAvailableActions(): list<string> {
        return ["archive", "edit"]
    }

    method archive(document: Document): void {
        document.setState(new ArchivedState())
        document.setArchivedAt(now())
        document.removeFromPublicAccess()
        log("Published document archived")
    }

    method edit(document: Document, content: string): void {
        // Create new draft version, keep published version
        newDraft = document.createNewVersion()
        newDraft.setContent(content)
        newDraft.setState(new DraftState())
        log("New draft version created from published document")
    }
}

// Archived state
class ArchivedState extends BaseDocumentState {
    method getName(): string {
        return "Archived"
    }

    method getAvailableActions(): list<string> {
        return ["restore"]
    }

    method restore(document: Document): void {
        document.setState(new DraftState())
        document.setRestoredAt(now())
        log("Document restored from archive")
    }
}
```

### Pseudocode: Context

```
class Document {
    private id: string
    private title: string
    private content: string
    private author: User
    private state: DocumentState
    private metadata: DocumentMetadata
    private history: list<StateTransition> = []

    constructor(title: string, author: User) {
        this.id = generateUUID()
        this.title = title
        this.author = author
        this.content = ""
        this.state = new DraftState()
        this.metadata = new DocumentMetadata()
    }

    // Delegate operations to current state
    method submit(): void {
        this.recordTransition("submit")
        this.state.submit(this)
    }

    method approve(): void {
        this.recordTransition("approve")
        this.state.approve(this)
    }

    method reject(): void {
        this.recordTransition("reject")
        this.state.reject(this)
    }

    method publish(): void {
        this.recordTransition("publish")
        this.state.publish(this)
    }

    method archive(): void {
        this.recordTransition("archive")
        this.state.archive(this)
    }

    method edit(content: string): void {
        this.recordTransition("edit")
        this.state.edit(this, content)
    }

    // State management
    method setState(state: DocumentState): void {
        previousState = this.state.getName()
        this.state = state
        this.history.add(new StateTransition(
            from: previousState,
            to: state.getName(),
            timestamp: now(),
            user: getCurrentUser()
        ))
    }

    method getState(): DocumentState {
        return this.state
    }

    method getStateName(): string {
        return this.state.getName()
    }

    method getAvailableActions(): list<string> {
        return this.state.getAvailableActions()
    }

    method canPerform(action: string): boolean {
        return action in this.getAvailableActions()
    }

    // Data accessors
    method getContent(): string {
        return this.content
    }

    method setContent(content: string): void {
        this.content = content
    }

    method getHistory(): list<StateTransition> {
        return this.history.copy()
    }

    // Notification helpers (called by states)
    method notifyReviewers(): void {
        reviewers = this.getAssignedReviewers()
        for reviewer in reviewers {
            notificationService.send(reviewer, "Document awaiting review: " + this.title)
        }
    }

    method notifyAuthor(action: string): void {
        notificationService.send(this.author, "Your document was " + action + ": " + this.title)
    }

    method notifySubscribers(): void {
        // Notify users subscribed to this topic/category
    }
}
```

## Example

A complete example implementing an order processing system:

```
// Order data
class OrderItem {
    public productId: string
    public productName: string
    public quantity: int
    public price: float

    method getSubtotal(): float {
        return this.quantity * this.price
    }
}

class ShippingInfo {
    public address: string
    public city: string
    public postalCode: string
    public country: string
    public method: string  // "standard", "express", "overnight"
}

class PaymentInfo {
    public method: string  // "credit_card", "paypal", "bank_transfer"
    public transactionId: string
    public paidAt: datetime
}

// Order state interface
interface OrderState {
    method pay(order: Order, payment: PaymentInfo): void
    method ship(order: Order, trackingNumber: string): void
    method deliver(order: Order): void
    method cancel(order: Order, reason: string): void
    method refund(order: Order): void
    method addItem(order: Order, item: OrderItem): void
    method removeItem(order: Order, productId: string): void

    method getName(): string
    method isFinal(): boolean
    method canModify(): boolean
    method canCancel(): boolean
}

// Abstract base state
abstract class BaseOrderState implements OrderState {
    method pay(order: Order, payment: PaymentInfo): void {
        throw new InvalidOperationException("Cannot pay in " + this.getName() + " state")
    }

    method ship(order: Order, trackingNumber: string): void {
        throw new InvalidOperationException("Cannot ship in " + this.getName() + " state")
    }

    method deliver(order: Order): void {
        throw new InvalidOperationException("Cannot deliver in " + this.getName() + " state")
    }

    method cancel(order: Order, reason: string): void {
        throw new InvalidOperationException("Cannot cancel in " + this.getName() + " state")
    }

    method refund(order: Order): void {
        throw new InvalidOperationException("Cannot refund in " + this.getName() + " state")
    }

    method addItem(order: Order, item: OrderItem): void {
        throw new InvalidOperationException("Cannot modify order in " + this.getName() + " state")
    }

    method removeItem(order: Order, productId: string): void {
        throw new InvalidOperationException("Cannot modify order in " + this.getName() + " state")
    }

    method isFinal(): boolean {
        return false
    }

    method canModify(): boolean {
        return false
    }

    method canCancel(): boolean {
        return false
    }
}

// Pending state - order created, awaiting payment
class PendingState extends BaseOrderState {
    private expirationTime: duration = Duration.ofHours(24)

    method getName(): string {
        return "Pending"
    }

    method canModify(): boolean {
        return true
    }

    method canCancel(): boolean {
        return true
    }

    method pay(order: Order, payment: PaymentInfo): void {
        // Validate payment
        if not this.validatePayment(order, payment) {
            throw new PaymentException("Payment validation failed")
        }

        // Process payment
        order.setPaymentInfo(payment)
        order.setState(new PaidState())
        order.recordEvent("Payment received: " + payment.transactionId)

        // Reserve inventory
        inventoryService.reserve(order.getItems())

        // Send confirmation
        emailService.sendOrderConfirmation(order)
    }

    method addItem(order: Order, item: OrderItem): void {
        if not inventoryService.isAvailable(item.productId, item.quantity) {
            throw new InventoryException("Product not available: " + item.productName)
        }

        order.addItemInternal(item)
        order.recordEvent("Item added: " + item.productName)
    }

    method removeItem(order: Order, productId: string): void {
        order.removeItemInternal(productId)
        order.recordEvent("Item removed: " + productId)
    }

    method cancel(order: Order, reason: string): void {
        order.setState(new CancelledState())
        order.setCancellationReason(reason)
        order.recordEvent("Order cancelled: " + reason)
    }

    private method validatePayment(order: Order, payment: PaymentInfo): boolean {
        // Validate payment amount matches order total
        return paymentGateway.validate(payment, order.getTotal())
    }

    // Check for expired pending orders
    method checkExpiration(order: Order): void {
        if now() > order.getCreatedAt().plus(this.expirationTime) {
            this.cancel(order, "Order expired - payment not received")
        }
    }
}

// Paid state - payment received, ready to ship
class PaidState extends BaseOrderState {
    method getName(): string {
        return "Paid"
    }

    method canCancel(): boolean {
        return true  // Can still cancel before shipping
    }

    method ship(order: Order, trackingNumber: string): void {
        // Validate shipping info
        if order.getShippingInfo() is null {
            throw new ShippingException("Shipping information required")
        }

        // Create shipment
        order.setTrackingNumber(trackingNumber)
        order.setShippedAt(now())
        order.setState(new ShippedState())
        order.recordEvent("Order shipped: " + trackingNumber)

        // Commit inventory (already reserved)
        inventoryService.commit(order.getItems())

        // Notify customer
        emailService.sendShippingNotification(order)
    }

    method cancel(order: Order, reason: string): void {
        // Release reserved inventory
        inventoryService.release(order.getItems())

        // Process refund
        refundResult = paymentGateway.refund(order.getPaymentInfo().transactionId)

        order.setRefundInfo(refundResult)
        order.setState(new CancelledState())
        order.setCancellationReason(reason)
        order.recordEvent("Order cancelled after payment: " + reason)

        emailService.sendCancellationNotification(order)
    }
}

// Shipped state - order is in transit
class ShippedState extends BaseOrderState {
    method getName(): string {
        return "Shipped"
    }

    method deliver(order: Order): void {
        order.setDeliveredAt(now())
        order.setState(new DeliveredState())
        order.recordEvent("Order delivered")

        emailService.sendDeliveryConfirmation(order)
    }

    method cancel(order: Order, reason: string): void {
        // Can't cancel shipped order - need to wait for delivery and refund
        throw new InvalidOperationException(
            "Cannot cancel shipped order. Please wait for delivery and request a refund."
        )
    }
}

// Delivered state - order received by customer
class DeliveredState extends BaseOrderState {
    private refundWindow: duration = Duration.ofDays(30)

    method getName(): string {
        return "Delivered"
    }

    method refund(order: Order): void {
        // Check refund window
        if now() > order.getDeliveredAt().plus(this.refundWindow) {
            throw new RefundException("Refund window has expired")
        }

        // Process refund
        refundResult = paymentGateway.refund(order.getPaymentInfo().transactionId)
        order.setRefundInfo(refundResult)
        order.setState(new RefundedState())
        order.recordEvent("Refund processed")

        // Note: inventory handling would depend on whether item is returned
        emailService.sendRefundConfirmation(order)
    }

    method isFinal(): boolean {
        // Can still be refunded, but typically final
        return true
    }
}

// Cancelled state - order was cancelled
class CancelledState extends BaseOrderState {
    method getName(): string {
        return "Cancelled"
    }

    method isFinal(): boolean {
        return true
    }
}

// Refunded state - order was refunded
class RefundedState extends BaseOrderState {
    method getName(): string {
        return "Refunded"
    }

    method isFinal(): boolean {
        return true
    }
}

// Context: Order
class Order {
    private id: string
    private customerId: string
    private items: list<OrderItem> = []
    private shippingInfo: ShippingInfo
    private paymentInfo: PaymentInfo
    private state: OrderState
    private createdAt: datetime
    private events: list<OrderEvent> = []

    // Shipping and delivery info
    private trackingNumber: string
    private shippedAt: datetime
    private deliveredAt: datetime

    // Cancellation/refund info
    private cancellationReason: string
    private refundInfo: RefundInfo

    constructor(customerId: string) {
        this.id = generateOrderId()
        this.customerId = customerId
        this.state = new PendingState()
        this.createdAt = now()
        this.recordEvent("Order created")
    }

    // Delegate to state
    method pay(payment: PaymentInfo): void {
        this.state.pay(this, payment)
    }

    method ship(trackingNumber: string): void {
        this.state.ship(this, trackingNumber)
    }

    method deliver(): void {
        this.state.deliver(this)
    }

    method cancel(reason: string): void {
        this.state.cancel(this, reason)
    }

    method refund(): void {
        this.state.refund(this)
    }

    method addItem(item: OrderItem): void {
        this.state.addItem(this, item)
    }

    method removeItem(productId: string): void {
        this.state.removeItem(this, productId)
    }

    // State management
    method setState(state: OrderState): void {
        previousState = this.state.getName()
        this.state = state
        this.recordEvent("State changed: " + previousState + " -> " + state.getName())
    }

    method getStateName(): string {
        return this.state.getName()
    }

    method canModify(): boolean {
        return this.state.canModify()
    }

    method canCancel(): boolean {
        return this.state.canCancel()
    }

    method isFinal(): boolean {
        return this.state.isFinal()
    }

    // Internal methods for states to use
    internal method addItemInternal(item: OrderItem): void {
        this.items.add(item)
    }

    internal method removeItemInternal(productId: string): void {
        this.items = this.items.filter(i => i.productId != productId)
    }

    // Calculations
    method getSubtotal(): float {
        return this.items.sum(item => item.getSubtotal())
    }

    method getTax(): float {
        return this.getSubtotal() * 0.08  // 8% tax
    }

    method getShippingCost(): float {
        if this.shippingInfo is null {
            return 0
        }
        switch this.shippingInfo.method {
            case "standard": return 5.99
            case "express": return 12.99
            case "overnight": return 24.99
            default: return 0
        }
    }

    method getTotal(): float {
        return this.getSubtotal() + this.getTax() + this.getShippingCost()
    }

    // Event tracking
    method recordEvent(description: string): void {
        this.events.add(new OrderEvent(
            timestamp: now(),
            description: description,
            state: this.state.getName()
        ))
    }

    method getEvents(): list<OrderEvent> {
        return this.events.copy()
    }

    // Getters/setters
    method getId(): string { return this.id }
    method getItems(): list<OrderItem> { return this.items.copy() }
    method getShippingInfo(): ShippingInfo { return this.shippingInfo }
    method setShippingInfo(info: ShippingInfo): void { this.shippingInfo = info }
    method getPaymentInfo(): PaymentInfo { return this.paymentInfo }
    method setPaymentInfo(info: PaymentInfo): void { this.paymentInfo = info }
    method getCreatedAt(): datetime { return this.createdAt }
    method getTrackingNumber(): string { return this.trackingNumber }
    method setTrackingNumber(num: string): void { this.trackingNumber = num }
    method getDeliveredAt(): datetime { return this.deliveredAt }
    method setDeliveredAt(dt: datetime): void { this.deliveredAt = dt }
    method getShippedAt(): datetime { return this.shippedAt }
    method setShippedAt(dt: datetime): void { this.shippedAt = dt }
    method setCancellationReason(reason: string): void { this.cancellationReason = reason }
    method setRefundInfo(info: RefundInfo): void { this.refundInfo = info }
}

// Usage example
function main() {
    // Create order
    order = new Order("customer-123")

    // Add items (only allowed in Pending state)
    order.addItem(new OrderItem("SKU-001", "Widget", 2, 29.99))
    order.addItem(new OrderItem("SKU-002", "Gadget", 1, 49.99))

    print("Order total: $" + order.getTotal())
    print("State: " + order.getStateName())  // "Pending"
    print("Can modify: " + order.canModify())  // true

    // Set shipping
    order.setShippingInfo(new ShippingInfo(
        address: "123 Main St",
        city: "Springfield",
        postalCode: "12345",
        country: "USA",
        method: "express"
    ))

    // Pay for order
    payment = new PaymentInfo(
        method: "credit_card",
        transactionId: "txn-456789"
    )
    order.pay(payment)
    print("State: " + order.getStateName())  // "Paid"
    print("Can modify: " + order.canModify())  // false

    // Try to add item (should fail)
    try {
        order.addItem(new OrderItem("SKU-003", "Thingy", 1, 19.99))
    } catch InvalidOperationException as e {
        print("Cannot modify: " + e.message)
    }

    // Ship order
    order.ship("TRACK-123456789")
    print("State: " + order.getStateName())  // "Shipped"

    // Deliver order
    order.deliver()
    print("State: " + order.getStateName())  // "Delivered"

    // Print order history
    print("\nOrder History:")
    for event in order.getEvents() {
        print(event.timestamp + " [" + event.state + "] " + event.description)
    }
}
```

## Known Uses

- **TCP Connection**: Connection states (LISTEN, SYN_SENT, ESTABLISHED, CLOSE_WAIT, etc.) are a classic state machine.

- **Java Thread**: Thread states (NEW, RUNNABLE, BLOCKED, WAITING, TERMINATED) in java.lang.Thread.

- **Document Workflows**: Word processors, CMS systems model document states (draft, review, published).

- **Game Development**: Character states (idle, walking, attacking, dead) in game engines.

- **Media Players**: Player states (stopped, playing, paused) in media applications.

- **Vending Machines**: Classic state machine example with states like NoCoin, HasCoin, Dispensing.

- **React Component Lifecycle**: Mount, update, unmount phases with corresponding methods.

- **Order Processing**: E-commerce order states in systems like Magento, Shopify.

- **Workflow Engines**: BPMN workflow states in tools like Camunda, Activiti.

## Related Patterns

- **Flyweight**: State objects can be shared as flyweights if they have no instance variables.

- **Singleton**: State objects are often singletons since only one instance is needed.

- **Strategy**: State can be considered a Strategy where the strategy is changed based on internal state rather than external configuration.

- **Bridge**: Both separate abstraction from implementation. State changes behavior based on internal state; Bridge separates interface hierarchy from implementation hierarchy.

## When NOT to Use

1. **Simple boolean flags**: If state is just true/false with minimal behavioral differences, a boolean is simpler.

```
// Overkill: State pattern for simple boolean
class EnabledState { method click() { doAction() } }
class DisabledState { method click() { /* nothing */ } }

// Just use a flag
method click() {
    if this.enabled {
        doAction()
    }
}
```

2. **Few states with simple behavior**: If you have 2-3 states with minimal state-specific logic, conditionals may be clearer.

3. **When states don't encapsulate behavior**: If the "state" is just data that other code interprets, you don't need state objects.

4. **Highly dynamic state definitions**: If states are defined at runtime from configuration, a data-driven state machine may be more appropriate than coded state classes.

5. **When transitions are the focus**: If transition rules are complex but state behavior is uniform, a state machine library or table-driven approach may be better.

```
// When transitions are complex, use a state machine definition
stateMachine = new StateMachine()
    .state("draft").on("submit").transitionTo("pending")
    .state("pending").on("approve").transitionTo("approved")
                     .on("reject").transitionTo("draft")
    // ... etc
```

6. **Performance-critical code**: The delegation overhead of the State pattern may be unacceptable in extremely hot paths.

The State pattern excels when objects have distinct behavioral modes with complex state-specific logic. Use simpler approaches for straightforward state tracking or when the overhead isn't justified.

---

*Based on concepts from "Design Patterns: Elements of Reusable Object-Oriented Software" by Gamma, Helm, Johnson, and Vlissides (Gang of Four), 1994.*
