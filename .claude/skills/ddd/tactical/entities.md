# Entities

## Definition

**Entities** are domain objects that have a distinct identity that runs through time and different representations. An entity is defined primarily by its identity, not by its attributes. Two entities with the same attribute values but different identities are considered different objects.

The identity of an Entity must be unique within a bounded context and remains constant throughout the object's lifecycle, even as its attributes change.

## Identity: Natural vs Surrogate Keys

### Natural Keys
Natural keys are identifiers derived from meaningful domain attributes:

```pseudocode
class Customer:
    email: Email  // Natural key - meaningful in the domain
    name: PersonName
    registrationDate: Date

    function identity():
        return this.email
```

**Characteristics:**
- Derived from business-meaningful attributes
- May already exist in the domain (SSN, email, ISBN)
- Can potentially change over time
- May have complex uniqueness constraints

### Surrogate Keys
Surrogate keys are artificial identifiers created solely for identification:

```pseudocode
class Order:
    orderId: OrderId  // Surrogate key - artificial identifier
    customerId: CustomerId
    orderDate: Date
    items: List<OrderItem>

    function identity():
        return this.orderId

class OrderId:
    value: UUID

    function equals(other: OrderId):
        return this.value == other.value
```

**Characteristics:**
- System-generated (UUIDs, sequences, GUIDs)
- Never change once assigned
- No business meaning
- Guaranteed uniqueness
- Simpler equality checks

**When to Use Each:**

| Scenario | Natural Key | Surrogate Key |
|----------|-------------|---------------|
| Identity exists in domain | ✓ | |
| Identity may change | | ✓ |
| Distributed systems | | ✓ |
| Simple equality needed | | ✓ |
| Business semantics important | ✓ | |
| Multiple potential identifiers | | ✓ |

## Equality Based on Identity

Entities are equal if and only if their identities are equal, regardless of attribute values:

```pseudocode
class User:
    userId: UserId
    username: String
    email: Email
    profile: UserProfile

    function equals(other: User):
        if other is null:
            return false
        return this.userId.equals(other.userId)

    function hashCode():
        return this.userId.hashCode()

// Usage example
user1 = new User(
    userId: UserId("123"),
    username: "john_doe",
    email: "john@example.com"
)

user2 = new User(
    userId: UserId("123"),
    username: "johnny",  // Different username
    email: "j.doe@example.com"  // Different email
)

user3 = new User(
    userId: UserId("456"),
    username: "john_doe",  // Same username as user1
    email: "john@example.com"  // Same email as user1
)

assert user1.equals(user2) == true   // Same identity
assert user1.equals(user3) == false  // Different identity
```

## Mutable State and Lifecycle

Unlike Value Objects, Entities are typically mutable and have a lifecycle:

```pseudocode
class Product:
    productId: ProductId
    name: ProductName
    price: Money
    inventory: InventoryLevel
    status: ProductStatus

    function identity():
        return this.productId

    // Lifecycle methods
    function activate():
        if this.status == ProductStatus.DRAFT:
            this.status = ProductStatus.ACTIVE
            raiseEvent(new ProductActivatedEvent(this.productId))

    function updatePrice(newPrice: Money):
        oldPrice = this.price
        this.price = newPrice
        raiseEvent(new PriceChangedEvent(
            productId: this.productId,
            oldPrice: oldPrice,
            newPrice: newPrice
        ))

    function adjustInventory(quantity: Integer):
        this.inventory = this.inventory.adjust(quantity)
        if this.inventory.isLow():
            raiseEvent(new LowInventoryEvent(this.productId))

    function discontinue():
        if this.status == ProductStatus.ACTIVE:
            this.status = ProductStatus.DISCONTINUED
            raiseEvent(new ProductDiscontinuedEvent(this.productId))

// Entity lifecycle
product = new Product(
    productId: ProductId.generate(),
    name: ProductName("Widget"),
    price: Money(19.99, "USD"),
    inventory: InventoryLevel(100),
    status: ProductStatus.DRAFT
)

product.activate()  // DRAFT -> ACTIVE
product.updatePrice(Money(24.99, "USD"))  // Price changes
product.adjustInventory(-10)  // Inventory changes
// Same product, but attributes have changed over time
```

## Entity vs Value Object Comparison

| Aspect | Entity | Value Object |
|--------|--------|--------------|
| **Identity** | Has unique identity | No identity |
| **Equality** | Based on identity | Based on attributes |
| **Mutability** | Typically mutable | Always immutable |
| **Lifecycle** | Has lifecycle events | No lifecycle |
| **Shareability** | Not safely shareable | Freely shareable |
| **Example** | User, Order, Product | Money, Address, DateRange |

```pseudocode
// Entity: Identity matters
order1 = new Order(orderId: "123", total: Money(100, "USD"))
order2 = new Order(orderId: "123", total: Money(200, "USD"))
assert order1.equals(order2) == true  // Same order, total changed

// Value Object: Attributes matter
money1 = new Money(100, "USD")
money2 = new Money(100, "USD")
money3 = new Money(200, "USD")
assert money1.equals(money2) == true   // Same value
assert money1.equals(money3) == false  // Different value
```

## Designing Entities

### 1. Identify Essential Identity
Determine what makes the entity unique in your domain:

```pseudocode
class Account:
    // Identity
    accountNumber: AccountNumber  // The essential identifier

    // Attributes (can change)
    balance: Money
    status: AccountStatus
    owner: CustomerId
    openedDate: Date
    lastModifiedDate: Date
```

### 2. Enforce Invariants
Entities must maintain their invariants throughout their lifecycle:

```pseudocode
class BankAccount:
    accountId: AccountId
    balance: Money
    overdraftLimit: Money
    status: AccountStatus

    function withdraw(amount: Money):
        // Enforce invariant: balance + overdraft >= amount
        if this.balance.add(this.overdraftLimit).isLessThan(amount):
            throw new InsufficientFundsException()

        // Enforce invariant: account must be active
        if this.status != AccountStatus.ACTIVE:
            throw new AccountNotActiveException()

        this.balance = this.balance.subtract(amount)
        raiseEvent(new WithdrawalMadeEvent(this.accountId, amount))
```

### 3. Manage Lifecycle Transitions
Model state transitions explicitly:

```pseudocode
class Subscription:
    subscriptionId: SubscriptionId
    status: SubscriptionStatus
    startDate: Date
    endDate: Date?

    function activate(startDate: Date):
        if this.status != SubscriptionStatus.PENDING:
            throw new InvalidStateTransitionException()

        this.status = SubscriptionStatus.ACTIVE
        this.startDate = startDate
        raiseEvent(new SubscriptionActivatedEvent(this.subscriptionId))

    function suspend():
        if this.status != SubscriptionStatus.ACTIVE:
            throw new InvalidStateTransitionException()

        this.status = SubscriptionStatus.SUSPENDED
        raiseEvent(new SubscriptionSuspendedEvent(this.subscriptionId))

    function cancel():
        if this.status in [SubscriptionStatus.CANCELLED, SubscriptionStatus.EXPIRED]:
            throw new InvalidStateTransitionException()

        this.status = SubscriptionStatus.CANCELLED
        this.endDate = Date.now()
        raiseEvent(new SubscriptionCancelledEvent(this.subscriptionId))
```

### 4. Encapsulate Collections
When an Entity contains collections, control access:

```pseudocode
class ShoppingCart:
    cartId: CartId
    customerId: CustomerId
    items: List<CartItem>  // Private collection

    function addItem(productId: ProductId, quantity: Integer):
        existingItem = this.findItem(productId)

        if existingItem != null:
            existingItem.increaseQuantity(quantity)
        else:
            this.items.add(new CartItem(productId, quantity))

        raiseEvent(new ItemAddedToCartEvent(this.cartId, productId, quantity))

    function removeItem(productId: ProductId):
        item = this.findItem(productId)
        if item == null:
            throw new ItemNotFoundException()

        this.items.remove(item)
        raiseEvent(new ItemRemovedFromCartEvent(this.cartId, productId))

    function getItems():
        // Return immutable view
        return this.items.asReadOnly()

    private function findItem(productId: ProductId):
        return this.items.find(item => item.productId == productId)
```

## Pseudocode Examples

### Example 1: User Entity

```pseudocode
class User:
    // Identity
    userId: UserId

    // Attributes
    username: Username
    email: Email
    passwordHash: PasswordHash
    profile: UserProfile
    roles: Set<Role>
    status: UserStatus
    createdAt: Timestamp
    lastLoginAt: Timestamp?

    function identity():
        return this.userId

    function equals(other: User):
        if other is null:
            return false
        return this.userId.equals(other.userId)

    function hashCode():
        return this.userId.hashCode()

    // Behavior
    function changeEmail(newEmail: Email):
        if this.email.equals(newEmail):
            return  // No change needed

        oldEmail = this.email
        this.email = newEmail
        raiseEvent(new EmailChangedEvent(
            userId: this.userId,
            oldEmail: oldEmail,
            newEmail: newEmail
        ))

    function changePassword(oldPassword: String, newPassword: String):
        if not this.passwordHash.matches(oldPassword):
            throw new InvalidPasswordException()

        if not PasswordPolicy.isValid(newPassword):
            throw new WeakPasswordException()

        this.passwordHash = PasswordHash.create(newPassword)
        raiseEvent(new PasswordChangedEvent(this.userId))

    function activate():
        if this.status == UserStatus.ACTIVE:
            return

        this.status = UserStatus.ACTIVE
        raiseEvent(new UserActivatedEvent(this.userId))

    function suspend(reason: String):
        if this.status == UserStatus.SUSPENDED:
            return

        this.status = UserStatus.SUSPENDED
        raiseEvent(new UserSuspendedEvent(this.userId, reason))

    function recordLogin():
        this.lastLoginAt = Timestamp.now()
        raiseEvent(new UserLoggedInEvent(this.userId))

    function assignRole(role: Role):
        if this.roles.contains(role):
            return

        this.roles.add(role)
        raiseEvent(new RoleAssignedEvent(this.userId, role))

    function hasPermission(permission: Permission):
        return this.roles.any(role => role.hasPermission(permission))
```

### Example 2: Order Entity

```pseudocode
class Order:
    // Identity
    orderId: OrderId

    // Attributes
    customerId: CustomerId
    orderDate: Date
    items: List<OrderItem>
    shippingAddress: Address
    billingAddress: Address
    status: OrderStatus
    paymentStatus: PaymentStatus
    totalAmount: Money

    function identity():
        return this.orderId

    function equals(other: Order):
        if other is null:
            return false
        return this.orderId.equals(other.orderId)

    // Factory method
    static function create(
        customerId: CustomerId,
        shippingAddress: Address,
        billingAddress: Address
    ):
        return new Order(
            orderId: OrderId.generate(),
            customerId: customerId,
            orderDate: Date.now(),
            items: [],
            shippingAddress: shippingAddress,
            billingAddress: billingAddress,
            status: OrderStatus.DRAFT,
            paymentStatus: PaymentStatus.PENDING,
            totalAmount: Money.zero("USD")
        )

    // Behavior
    function addItem(productId: ProductId, quantity: Integer, unitPrice: Money):
        if this.status != OrderStatus.DRAFT:
            throw new OrderNotModifiableException()

        existingItem = this.findItem(productId)
        if existingItem != null:
            existingItem.increaseQuantity(quantity)
        else:
            this.items.add(new OrderItem(productId, quantity, unitPrice))

        this.recalculateTotal()
        raiseEvent(new ItemAddedToOrderEvent(this.orderId, productId, quantity))

    function removeItem(productId: ProductId):
        if this.status != OrderStatus.DRAFT:
            throw new OrderNotModifiableException()

        item = this.findItem(productId)
        if item == null:
            throw new ItemNotFoundException()

        this.items.remove(item)
        this.recalculateTotal()
        raiseEvent(new ItemRemovedFromOrderEvent(this.orderId, productId))

    function submit():
        if this.status != OrderStatus.DRAFT:
            throw new InvalidOrderStateException()

        if this.items.isEmpty():
            throw new EmptyOrderException()

        this.status = OrderStatus.SUBMITTED
        raiseEvent(new OrderSubmittedEvent(this.orderId, this.totalAmount))

    function confirmPayment():
        if this.paymentStatus == PaymentStatus.PAID:
            return

        this.paymentStatus = PaymentStatus.PAID
        raiseEvent(new PaymentConfirmedEvent(this.orderId))

        if this.canFulfill():
            this.fulfill()

    function fulfill():
        if this.status != OrderStatus.SUBMITTED:
            throw new InvalidOrderStateException()

        if this.paymentStatus != PaymentStatus.PAID:
            throw new PaymentNotConfirmedException()

        this.status = OrderStatus.FULFILLED
        raiseEvent(new OrderFulfilledEvent(this.orderId))

    function cancel(reason: String):
        if this.status in [OrderStatus.FULFILLED, OrderStatus.CANCELLED]:
            throw new OrderNotCancellableException()

        this.status = OrderStatus.CANCELLED
        raiseEvent(new OrderCancelledEvent(this.orderId, reason))

    private function recalculateTotal():
        this.totalAmount = this.items
            .map(item => item.getLineTotal())
            .reduce(Money.zero("USD"), (sum, lineTotal) => sum.add(lineTotal))

    private function findItem(productId: ProductId):
        return this.items.find(item => item.productId.equals(productId))

    private function canFulfill():
        return this.status == OrderStatus.SUBMITTED and
               this.paymentStatus == PaymentStatus.PAID
```

### Example 3: Product Entity

```pseudocode
class Product:
    // Identity
    productId: ProductId

    // Attributes
    sku: SKU
    name: ProductName
    description: String
    category: CategoryId
    price: Money
    cost: Money
    inventory: InventoryLevel
    status: ProductStatus
    createdAt: Timestamp
    lastModifiedAt: Timestamp

    function identity():
        return this.productId

    function equals(other: Product):
        if other is null:
            return false
        return this.productId.equals(other.productId)

    // Factory
    static function create(
        sku: SKU,
        name: ProductName,
        category: CategoryId,
        price: Money,
        cost: Money
    ):
        return new Product(
            productId: ProductId.generate(),
            sku: sku,
            name: name,
            description: "",
            category: category,
            price: price,
            cost: cost,
            inventory: InventoryLevel.zero(),
            status: ProductStatus.DRAFT,
            createdAt: Timestamp.now(),
            lastModifiedAt: Timestamp.now()
        )

    // Behavior
    function updatePrice(newPrice: Money):
        if newPrice.isLessThanOrEqual(Money.zero(newPrice.currency)):
            throw new InvalidPriceException()

        if newPrice.isLessThan(this.cost):
            raiseWarning(new PriceBelowCostWarning(this.productId))

        oldPrice = this.price
        this.price = newPrice
        this.lastModifiedAt = Timestamp.now()
        raiseEvent(new PriceChangedEvent(this.productId, oldPrice, newPrice))

    function adjustInventory(adjustment: Integer, reason: String):
        newLevel = this.inventory.adjust(adjustment)

        if newLevel.isNegative():
            throw new NegativeInventoryException()

        oldLevel = this.inventory
        this.inventory = newLevel
        this.lastModifiedAt = Timestamp.now()

        raiseEvent(new InventoryAdjustedEvent(
            productId: this.productId,
            oldLevel: oldLevel,
            newLevel: newLevel,
            reason: reason
        ))

        if newLevel.isLow():
            raiseEvent(new LowInventoryAlertEvent(this.productId, newLevel))

    function activate():
        if this.status == ProductStatus.ACTIVE:
            return

        this.validateForActivation()
        this.status = ProductStatus.ACTIVE
        this.lastModifiedAt = Timestamp.now()
        raiseEvent(new ProductActivatedEvent(this.productId))

    function discontinue():
        if this.status == ProductStatus.DISCONTINUED:
            return

        this.status = ProductStatus.DISCONTINUED
        this.lastModifiedAt = Timestamp.now()
        raiseEvent(new ProductDiscontinuedEvent(this.productId))

    function updateDescription(newDescription: String):
        this.description = newDescription
        this.lastModifiedAt = Timestamp.now()
        raiseEvent(new ProductDescriptionUpdatedEvent(this.productId))

    function isAvailable():
        return this.status == ProductStatus.ACTIVE and
               this.inventory.isAvailable()

    function getMargin():
        return this.price.subtract(this.cost)

    function getMarginPercentage():
        if this.price.isZero():
            return 0
        return (this.getMargin().amount / this.price.amount) * 100

    private function validateForActivation():
        if this.name.isEmpty():
            throw new ValidationException("Product name required")

        if this.price.isLessThanOrEqual(Money.zero(this.price.currency)):
            throw new ValidationException("Valid price required")

        if this.category == null:
            throw new ValidationException("Category required")
```

## Summary Table

| Concept | Description | Example |
|---------|-------------|---------|
| **Identity** | Unique identifier that persists through time | `userId`, `orderId`, `productId` |
| **Natural Key** | Identity from domain-meaningful attributes | Email, ISBN, SSN |
| **Surrogate Key** | Artificial identifier (UUID, sequence) | System-generated IDs |
| **Equality** | Based on identity, not attributes | Same ID = same entity |
| **Mutability** | State changes over lifecycle | Price updates, status changes |
| **Lifecycle** | Transitions through states | Draft → Active → Discontinued |
| **Invariants** | Rules maintained throughout lifecycle | Balance + overdraft >= 0 |
| **Encapsulation** | Controlled state changes via methods | `activate()`, `updatePrice()` |
| **Events** | Domain events on state changes | `ProductActivatedEvent` |
| **Collections** | Managed through entity methods | `addItem()`, `removeItem()` |

---

*Based on concepts from "Domain-Driven Design" by Eric Evans.*
