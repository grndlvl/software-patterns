# Aggregates

## Definition

An **Aggregate** is a cluster of associated objects (entities and value objects) that are treated as a single unit with respect to data changes. An Aggregate enforces a strong consistency boundary around a set of related domain objects and maintains a set of invariants—rules that must always be true.

The Aggregate is the fundamental building block for modeling domain logic in DDD. It defines a transactional consistency boundary: changes to objects within an Aggregate must be atomic and consistent. Objects outside the Aggregate interact with it only through its public interface.

**Key Characteristics:**
- Clusters entities and value objects with consistent behavior
- Has a single entry point: the Aggregate Root
- Maintains invariants across all contained objects
- Operates as a unit of consistency
- Referenced by other aggregates through its root's identity only

## Aggregate Root Concept

The **Aggregate Root** is the entry point to an Aggregate. It is an Entity that serves as the public interface through which external objects interact with the entire Aggregate. The Aggregate Root has the following responsibilities:

1. **Enforce Invariants**: Ensure all rules are maintained for all objects in the Aggregate
2. **Single Entry Point**: All external references come through the Root only
3. **Transactional Boundary**: Changes to the Aggregate are atomic
4. **Identity Management**: Provides the identity by which the Aggregate is known externally
5. **Lifecycle Management**: Controls the lifecycle of all contained objects

### Aggregate Root Example

```pseudocode
class Order:  // Aggregate Root
    orderId: OrderId
    customerId: CustomerId
    items: List<OrderItem>  // Internal collection
    status: OrderStatus
    totalAmount: Money

    // Aggregate Root enforces invariants
    function addItem(productId: ProductId, quantity: Integer, unitPrice: Money):
        // Only OrderItem can be added; external code cannot directly modify items
        if this.status != OrderStatus.DRAFT:
            throw new OrderNotModifiableException()

        item = new OrderItem(productId, quantity, unitPrice)
        this.items.add(item)
        this.recalculateTotal()

    // Return immutable reference
    function getItems():
        return this.items.asReadOnly()

    // Direct access to items NOT allowed
    // function getItemsList(): // NOT PROVIDED
```

### What Cannot be in the Aggregate Root Interface

- Direct collection access: `aggregate.items.add(...)` (WRONG)
- Direct property modification: `aggregate.status = ...` (WRONG)
- Unvalidated state changes: No invariant enforcement (WRONG)

### What Can be in the Aggregate Root Interface

- Behavioral methods: `addItem(...)`, `removeItem(...)` (RIGHT)
- Query methods: `getItems()`, `getTotalAmount()` (RIGHT)
- Lifecycle transitions: `submit()`, `confirm()`, `cancel()` (RIGHT)

## Design Rules

### Rule 1: Small Aggregates

Keep Aggregates small and focused. An Aggregate should contain only what is necessary to enforce its invariants.

```pseudocode
// WRONG: Too large, multiple consistency boundaries
class Customer:
    customerId: CustomerId
    name: PersonName
    addresses: List<Address>
    orders: List<Order>  // Orders have their own invariants!
    shipments: List<Shipment>
    invoices: List<Invoice>
    reviews: List<Review>

// RIGHT: Focused Aggregates
class Customer:  // Aggregate Root
    customerId: CustomerId
    name: PersonName
    addresses: List<Address>  // Part of Customer invariants
    preferredAddress: Address

class Order:  // Separate Aggregate Root
    orderId: OrderId
    customerId: CustomerId  // Reference to Customer, not the object
    items: List<OrderItem>
    status: OrderStatus

class Shipment:  // Separate Aggregate Root
    shipmentId: ShipmentId
    orderId: OrderId  // Reference to Order, not the object
    status: ShipmentStatus

class Invoice:  // Separate Aggregate Root
    invoiceId: InvoiceId
    orderId: OrderId  // Reference to Order, not the object
    amount: Money
```

### Rule 2: Reference Other Aggregates by Identity Only

Do not hold direct references to other Aggregate Roots. Instead, store only their identities.

```pseudocode
// WRONG: Direct reference to another Aggregate Root
class Order:
    customer: Customer  // WRONG - now Order depends on Customer internals
    items: List<OrderItem>

// RIGHT: Reference by identity
class Order:
    customerId: CustomerId  // Reference by ID only
    items: List<OrderItem>

// To access the Customer, fetch from repository
order = orderRepository.getById(orderId)
customer = customerRepository.getById(order.customerId)
```

**Exception**: Small Value Objects can be embedded. They are immutable and don't have their own identity.

```pseudocode
class Order:
    customerId: CustomerId  // Identity reference
    shippingAddress: Address  // Value Object - okay to embed
    billingAddress: Address  // Value Object - okay to embed
    items: List<OrderItem>  // Internal entities
```

### Rule 3: One Aggregate per Transaction

A single transaction should modify at most one Aggregate Root. If multiple Aggregates need to change, use Domain Events and eventual consistency.

```pseudocode
// WRONG: Modifying two Aggregates in one transaction
function transferFunds(fromAccountId: AccountId, toAccountId: AccountId, amount: Money):
    fromAccount = accountRepository.getById(fromAccountId)
    toAccount = accountRepository.getById(toAccountId)

    // Modifying TWO aggregates - bad!
    fromAccount.withdraw(amount)
    toAccount.deposit(amount)

    accountRepository.save(fromAccount)
    accountRepository.save(toAccount)
    // If second save fails, we're in an inconsistent state

// RIGHT: Modify one Aggregate, publish event
function transferFunds(fromAccountId: AccountId, toAccountId: AccountId, amount: Money):
    fromAccount = accountRepository.getById(fromAccountId)
    fromAccount.withdraw(amount)  // Modify ONE Aggregate
    accountRepository.save(fromAccount)

    // Event handler will eventually process
    // This is ASYNCHRONOUS and eventual consistency
    // publishEvent(new FundsTransferredEvent(fromAccountId, toAccountId, amount))

// Elsewhere: Event handler
class FundsTransferredEventHandler:
    function handle(event: FundsTransferredEvent):
        toAccount = accountRepository.getById(event.toAccountId)
        toAccount.deposit(event.amount)
        accountRepository.save(toAccount)
```

## Pseudocode Example: Order Aggregate

```pseudocode
// ==================== VALUE OBJECTS ====================

class OrderItemId:
    value: UUID

    function equals(other: OrderItemId):
        return this.value == other.value

class ProductId:
    value: UUID

    function equals(other: ProductId):
        return this.value == other.value

class Quantity:
    value: Integer

    function validate():
        if this.value <= 0:
            throw new InvalidQuantityException()

    function increment(delta: Integer):
        return new Quantity(this.value + delta)

    function decrement(delta: Integer):
        if this.value - delta <= 0:
            throw new InvalidQuantityException()
        return new Quantity(this.value - delta)

    function equals(other: Quantity):
        return this.value == other.value

class LineTotal:
    amount: Money

    function equals(other: LineTotal):
        return this.amount.equals(other.amount)

// ==================== INTERNAL ENTITY ====================

class OrderItem:  // Internal Entity - NOT an Aggregate Root
    itemId: OrderItemId
    productId: ProductId
    quantity: Quantity
    unitPrice: Money

    function identity():
        return this.itemId

    function getTotalAmount():
        return this.unitPrice.multiply(this.quantity.value)

    function updateQuantity(newQuantity: Quantity):
        newQuantity.validate()
        this.quantity = newQuantity

    function equals(other: OrderItem):
        return this.itemId.equals(other.itemId)

// ==================== AGGREGATE ROOT ====================

class Order:  // Aggregate Root
    orderId: OrderId
    customerId: CustomerId
    orderDate: Date
    items: List<OrderItem>  // Internal collection
    shippingAddress: Address  // Value Object
    billingAddress: Address  // Value Object
    status: OrderStatus
    paymentStatus: PaymentStatus
    createdAt: Timestamp
    modifiedAt: Timestamp

    // ========== AGGREGATE IDENTITY ==========
    function identity():
        return this.orderId

    function equals(other: Order):
        if other is null:
            return false
        return this.orderId.equals(other.orderId)

    // ========== FACTORY ==========
    static function create(
        customerId: CustomerId,
        shippingAddress: Address,
        billingAddress: Address
    ):
        order = new Order(
            orderId: OrderId.generate(),
            customerId: customerId,
            orderDate: Date.now(),
            items: [],
            shippingAddress: shippingAddress,
            billingAddress: billingAddress,
            status: OrderStatus.DRAFT,
            paymentStatus: PaymentStatus.PENDING,
            createdAt: Timestamp.now(),
            modifiedAt: Timestamp.now()
        )

        order.raiseEvent(new OrderCreatedEvent(
            orderId: order.orderId,
            customerId: customerId
        ))

        return order

    // ========== INVARIANT ENFORCEMENT ==========

    // Invariant 1: Only DRAFT orders can be modified
    private function ensureDraftStatus():
        if this.status != OrderStatus.DRAFT:
            throw new OrderNotModifiableException(
                "Cannot modify order in " + this.status + " status"
            )

    // Invariant 2: Order must have at least one item before submission
    private function ensureHasItems():
        if this.items.isEmpty():
            throw new EmptyOrderException("Order must contain at least one item")

    // Invariant 3: Items must be unique by product
    private function ensureProductUniqueness(productId: ProductId):
        existingItem = this.findItem(productId)
        if existingItem != null:
            throw new DuplicateProductException(productId)

    // Invariant 4: Payment must be confirmed before fulfillment
    private function ensurePaymentConfirmed():
        if this.paymentStatus != PaymentStatus.PAID:
            throw new PaymentNotConfirmedException()

    // ========== AGGREGATE BEHAVIOR ==========

    function addItem(productId: ProductId, quantity: Quantity, unitPrice: Money):
        // Enforce invariant: only modify DRAFT orders
        this.ensureDraftStatus()

        // Validate inputs
        quantity.validate()
        if unitPrice.isNegative():
            throw new InvalidPriceException()

        // Enforce invariant: unique products
        this.ensureProductUniqueness(productId)

        // Create and add item
        item = new OrderItem(
            itemId: OrderItemId.generate(),
            productId: productId,
            quantity: quantity,
            unitPrice: unitPrice
        )

        this.items.add(item)
        this.modifiedAt = Timestamp.now()

        // Raise domain event
        this.raiseEvent(new ItemAddedToOrderEvent(
            orderId: this.orderId,
            productId: productId,
            quantity: quantity,
            unitPrice: unitPrice
        ))

    function removeItem(productId: ProductId):
        // Enforce invariant: only modify DRAFT orders
        this.ensureDraftStatus()

        item = this.findItem(productId)
        if item == null:
            throw new ItemNotFoundException(productId)

        this.items.remove(item)
        this.modifiedAt = Timestamp.now()

        this.raiseEvent(new ItemRemovedFromOrderEvent(
            orderId: this.orderId,
            productId: productId
        ))

    function updateItemQuantity(productId: ProductId, newQuantity: Quantity):
        // Enforce invariant: only modify DRAFT orders
        this.ensureDraftStatus()

        item = this.findItem(productId)
        if item == null:
            throw new ItemNotFoundException(productId)

        oldQuantity = item.quantity
        item.updateQuantity(newQuantity)
        this.modifiedAt = Timestamp.now()

        this.raiseEvent(new ItemQuantityUpdatedEvent(
            orderId: this.orderId,
            productId: productId,
            oldQuantity: oldQuantity,
            newQuantity: newQuantity
        ))

    function submit():
        // Enforce invariant: order must be DRAFT
        if this.status != OrderStatus.DRAFT:
            throw new InvalidOrderStateException()

        // Enforce invariant: must have items
        this.ensureHasItems()

        this.status = OrderStatus.SUBMITTED
        this.modifiedAt = Timestamp.now()

        this.raiseEvent(new OrderSubmittedEvent(
            orderId: this.orderId,
            customerId: this.customerId,
            totalAmount: this.getTotalAmount()
        ))

    function confirmPayment():
        // Enforce invariant: order must be SUBMITTED
        if this.status != OrderStatus.SUBMITTED:
            throw new InvalidOrderStateException()

        this.paymentStatus = PaymentStatus.PAID
        this.modifiedAt = Timestamp.now()

        this.raiseEvent(new PaymentConfirmedEvent(
            orderId: this.orderId,
            amount: this.getTotalAmount()
        ))

        // Auto-transition to FULFILLED if payment confirmed
        if this.canBeFulfilled():
            this.fulfill()

    function fulfill():
        // Enforce invariant: order must be SUBMITTED
        if this.status != OrderStatus.SUBMITTED:
            throw new InvalidOrderStateException()

        // Enforce invariant: payment must be confirmed
        this.ensurePaymentConfirmed()

        this.status = OrderStatus.FULFILLED
        this.modifiedAt = Timestamp.now()

        this.raiseEvent(new OrderFulfilledEvent(
            orderId: this.orderId,
            customerId: this.customerId
        ))

    function cancel(reason: String):
        // Cannot cancel fulfilled or already cancelled orders
        if this.status in [OrderStatus.FULFILLED, OrderStatus.CANCELLED]:
            throw new OrderNotCancellableException()

        this.status = OrderStatus.CANCELLED
        this.modifiedAt = Timestamp.now()

        this.raiseEvent(new OrderCancelledEvent(
            orderId: this.orderId,
            customerId: this.customerId,
            reason: reason
        ))

    // ========== QUERIES (No side effects) ==========

    function getTotalAmount():
        total = Money.zero("USD")

        for each item in this.items:
            total = total.add(item.getTotalAmount())

        return total

    function getItems():
        // Return immutable view - external code cannot modify
        return this.items.asReadOnly()

    function getItemCount():
        return this.items.size()

    function hasItem(productId: ProductId):
        return this.findItem(productId) != null

    function getItemQuantity(productId: ProductId):
        item = this.findItem(productId)
        return item != null ? item.quantity : null

    function canBeFulfilled():
        return this.status == OrderStatus.SUBMITTED and
               this.paymentStatus == PaymentStatus.PAID

    function isModifiable():
        return this.status == OrderStatus.DRAFT

    // ========== PRIVATE HELPERS ==========

    private function findItem(productId: ProductId):
        return this.items.find(item => item.productId.equals(productId))

    private function raiseEvent(event: DomainEvent):
        // Event will be persisted and published by repository
        this.domainEvents.add(event)
```

## Invariant Enforcement Examples

### Example 1: Bank Account Aggregate

```pseudocode
class BankAccount:
    accountId: AccountId
    balance: Money
    overdraftLimit: Money
    status: AccountStatus
    dailyWithdrawalLimit: Money
    dailyWithdrawnToday: Money

    // ========== INVARIANTS ==========
    // Invariant 1: balance + overdraft >= 0
    // Invariant 2: account must be ACTIVE to withdraw
    // Invariant 3: daily withdrawals <= daily limit
    // Invariant 4: overdraft limit cannot be negative

    function withdraw(amount: Money):
        // Enforce invariant 1: sufficient funds
        availableFunds = this.balance.add(this.overdraftLimit)
        if availableFunds.isLessThan(amount):
            throw new InsufficientFundsException(
                "Available: " + availableFunds + ", Requested: " + amount
            )

        // Enforce invariant 2: account must be active
        if this.status != AccountStatus.ACTIVE:
            throw new AccountNotActiveException()

        // Enforce invariant 3: daily limit
        if this.dailyWithdrawnToday.add(amount).isGreaterThan(this.dailyWithdrawalLimit):
            throw new DailyLimitExceededException()

        this.balance = this.balance.subtract(amount)
        this.dailyWithdrawnToday = this.dailyWithdrawnToday.add(amount)

        this.raiseEvent(new WithdrawalMadeEvent(
            accountId: this.accountId,
            amount: amount,
            newBalance: this.balance
        ))

    function setOverdraftLimit(newLimit: Money):
        // Enforce invariant 4: no negative overdraft
        if newLimit.isNegative():
            throw new InvalidOverdraftLimitException()

        oldLimit = this.overdraftLimit
        this.overdraftLimit = newLimit

        this.raiseEvent(new OverdraftLimitChangedEvent(
            accountId: this.accountId,
            oldLimit: oldLimit,
            newLimit: newLimit
        ))

    function deposit(amount: Money):
        if amount.isLessThanOrEqual(Money.zero("USD")):
            throw new InvalidAmountException()

        this.balance = this.balance.add(amount)

        this.raiseEvent(new DepositMadeEvent(
            accountId: this.accountId,
            amount: amount,
            newBalance: this.balance
        ))

    function resetDailyWithdrawalCounter():
        this.dailyWithdrawnToday = Money.zero(this.balance.currency)
```

### Example 2: Reservation Aggregate

```pseudocode
class Reservation:
    reservationId: ReservationId
    customerId: CustomerId
    resourceId: ResourceId
    startTime: DateTime
    endTime: DateTime
    status: ReservationStatus
    confirmedAt: Timestamp?

    // ========== INVARIANTS ==========
    // Invariant 1: startTime < endTime
    // Invariant 2: cannot confirm CANCELLED reservations
    // Invariant 3: can only confirm PENDING reservations
    // Invariant 4: confirmation deadline must not pass

    function confirm(confirmationDeadline: DateTime):
        // Enforce invariant 2: not CANCELLED
        if this.status == ReservationStatus.CANCELLED:
            throw new ReservationCancelledException()

        // Enforce invariant 3: must be PENDING
        if this.status != ReservationStatus.PENDING:
            throw new ReservationAlreadyConfirmedException()

        // Enforce invariant 4: within deadline
        if DateTime.now().isAfter(confirmationDeadline):
            throw new ConfirmationDeadlinePassedException()

        this.status = ReservationStatus.CONFIRMED
        this.confirmedAt = Timestamp.now()

        this.raiseEvent(new ReservationConfirmedEvent(
            reservationId: this.reservationId,
            resourceId: this.resourceId
        ))

    function reschedule(newStartTime: DateTime, newEndTime: DateTime):
        // Enforce invariant 3: can only reschedule PENDING
        if this.status != ReservationStatus.PENDING:
            throw new CannotRescheduleConfirmedReservation()

        // Enforce invariant 1: times valid
        if newStartTime.isAfterOrEqual(newEndTime):
            throw new InvalidTimeRangeException()

        oldStart = this.startTime
        oldEnd = this.endTime

        this.startTime = newStartTime
        this.endTime = newEndTime

        this.raiseEvent(new ReservationRescheduledEvent(
            reservationId: this.reservationId,
            oldStartTime: oldStart,
            oldEndTime: oldEnd,
            newStartTime: newStartTime,
            newEndTime: newEndTime
        ))

    function cancel(reason: String):
        // Cannot cancel already cancelled
        if this.status == ReservationStatus.CANCELLED:
            return

        this.status = ReservationStatus.CANCELLED

        this.raiseEvent(new ReservationCancelledEvent(
            reservationId: this.reservationId,
            reason: reason
        ))
```

## Table: Aggregate Design Guidelines

| Aspect | Guidelines |
|--------|------------|
| **Size** | Keep small; one consistency boundary per Aggregate |
| **Root** | Designate single Entity as Aggregate Root |
| **Internal Objects** | Entities and Value Objects within boundary |
| **External References** | Reference other Aggregates by ID only, never by object reference |
| **Invariants** | Root enforces all invariants across contained objects |
| **Transactions** | One Aggregate Root per transaction |
| **Events** | Publish Domain Events for cross-Aggregate concerns |
| **Collections** | Expose through Root methods only, never direct access |
| **Immutability** | Favor immutable Value Objects within Aggregate |
| **Identity** | Root has identity; internal Entities may have local identity |
| **Lifecycles** | Root controls lifecycle of contained objects |
| **Queries** | Query methods on Root for reading aggregate state |
| **Commands** | Behavioral methods on Root for state changes |

## Aggregate vs Bounded Context

A **Bounded Context** may contain multiple Aggregates. The relationship:

```pseudocode
// Bounded Context: Sales Management
class SalesContext:
    // Multiple Aggregates within one Bounded Context
    
    // Aggregate 1: Customer Aggregate
    class Customer:
        customerId: CustomerId
        name: PersonName
        addresses: List<Address>

    // Aggregate 2: Order Aggregate
    class Order:
        orderId: OrderId
        customerId: CustomerId  // Reference to Customer Aggregate
        items: List<OrderItem>

    // Aggregate 3: Quote Aggregate
    class Quote:
        quoteId: QuoteId
        customerId: CustomerId  // Reference to Customer Aggregate
        lineItems: List<QuoteLineItem>

// Cross-Context Communication
class OrderService:
    function createOrderFromQuote(quoteId: QuoteId):
        quote = quoteRepository.getById(quoteId)  // From Quote Aggregate

        order = Order.create(quote.customerId)

        for each lineItem in quote.lineItems:
            order.addItem(lineItem.productId, lineItem.quantity, lineItem.price)

        orderRepository.save(order)  // Save Order Aggregate
        publishEvent(new OrderCreatedEvent(order.orderId))  // Event for other Contexts
```

## Summary Table

| Concept | Purpose | Example |
|---------|---------|---------|
| **Aggregate** | Cluster of related objects with consistent behavior | Order with OrderItems |
| **Aggregate Root** | Entry point and consistency enforcer | Order (not OrderItem) |
| **Identity Reference** | External reference without object coupling | `customerId` instead of `customer` |
| **Invariant** | Rule that must always be true | Balance + Overdraft >= 0 |
| **Factory** | Create valid Aggregate instances | `Order.create(...)` |
| **Behavioral Method** | Modify state while enforcing invariants | `addItem()`, `withdraw()` |
| **Value Object** | Immutable object with no identity | Money, Address, Quantity |
| **Internal Entity** | Entity without external identity | OrderItem, ReservationLine |
| **Domain Event** | Fact that happened in the domain | `OrderSubmittedEvent` |
| **Repository** | Persist and retrieve Aggregates | `orderRepository.save(order)` |
| **Transactional Boundary** | Atomic unit of change | One Aggregate per transaction |

---

*Based on concepts from "Domain-Driven Design" by Eric Evans.*
