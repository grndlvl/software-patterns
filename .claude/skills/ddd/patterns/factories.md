# DDD Factories

## Definition

A Factory is a pattern for encapsulating the complex logic required to create an aggregate or entity. Factories abstract away the details of object construction, ensuring that complex invariants are enforced at creation time and that newly created objects are always in a valid state.

In Domain-Driven Design, factories move creation logic out of domain classes and consolidate it into a dedicated, stateless service that understands the aggregate's internal structure.

---

## When to Use Factories

### Use Factories When:

- **Complex Aggregates**: The aggregate requires multiple steps to initialize or has complex interdependencies between parts
- **Invariant Enforcement**: Invariants must be satisfied immediately upon creation (not after construction)
- **Encapsulation**: Internal aggregate structure should not be exposed to clients
- **Validation Rules**: Creation involves domain-specific validation beyond simple type checking
- **Configuration Complexity**: Multiple valid configurations exist with different initialization paths
- **Reconstitution**: Objects need to be recreated from persistent storage in a safe, consistent way

### Don't Use Factories When:

- Simple value objects or entities with straightforward constructors
- Creation logic is trivial (just assigning fields)
- No invariants to enforce during creation
- Client code doesn't need abstraction

---

## Factory Method vs Abstract Factory in DDD

### Factory Method (Single Aggregate)

**When**: Creating a single type of aggregate with multiple initialization strategies.

**Example**: `OrderFactory.createFromCustomer()` or `OrderFactory.createFromQuote()`

Provides **one factory class with multiple methods**, each handling a different creation scenario.

```
interface OrderFactory
  method createFromCustomer(customer, items) -> Order
  method createFromQuote(quote) -> Order
  method createFromTemplate(template, modifications) -> Order
```

**Strengths**:
- Keeps related creation methods together
- Clear responsibility: one factory per aggregate type
- Easy to test

**Weaknesses**:
- All methods tightly coupled to Order details
- Harder to swap implementations

---

### Abstract Factory (Family of Related Aggregates)

**When**: Creating multiple related aggregates that must be consistent with each other (e.g., Order + Invoice + Shipment).

**Example**: `EcommerceFactory.createOrder()`, `createInvoice()`, `createShipment()`

Provides **multiple factory methods** for creating a family of related objects.

```
interface OrderManagementFactory
  method createOrder(customer, items) -> Order
  method createInvoice(order) -> Invoice
  method createShipment(order) -> Shipment

class StandardOrderFactory implements OrderManagementFactory
  ...

class B2BOrderFactory implements OrderManagementFactory
  ...
```

**Strengths**:
- Ensures consistency across related aggregates
- Easy to swap entire families (e.g., B2B vs B2C behavior)
- Centralizes related creation logic

**Weaknesses**:
- More complex, adds indirection
- Overkill for simple scenarios

---

## Pseudocode: OrderFactory Creating a Complete Aggregate

### Simple Factory Method

```pseudocode
class OrderFactory
  method createFromCustomer(customerId, lineItems) -> Order
    // Validate inputs
    if lineItems.isEmpty()
      throw InvalidOrderException("Order must have at least one item")
    
    // Load customer aggregate
    customer = customerRepository.findById(customerId)
    if customer is null
      throw CustomerNotFoundException(customerId)
    
    // Create order aggregate with all parts
    order = new Order(
      id: OrderId.generate(),
      customerId: customer.id(),
      status: OrderStatus.PENDING,
      createdAt: now(),
      lines: createOrderLines(lineItems),
      subtotal: calculateSubtotal(lineItems),
      shippingAddress: customer.defaultShippingAddress(),
      total: 0  // Will be set below
    )
    
    // Apply business rules
    applyTaxRules(order, customer.taxJurisdiction())
    applyDiscount(order, customer.loyaltyTier())
    order.recalculateTotal()
    
    // Verify invariants before returning
    if not order.isValid()
      throw InvalidAggregateException("Order failed invariant checks")
    
    return order
  
  method createOrderLines(lineItemDTOs) -> List[OrderLine]
    lines = []
    for each dto in lineItemDTOs
      // Validate each line
      if dto.quantity <= 0
        throw InvalidQuantityException(dto.quantity)
      
      product = productRepository.findById(dto.productId)
      if product is null
        throw ProductNotFoundException(dto.productId)
      
      // Create line as value object
      line = new OrderLine(
        productId: product.id(),
        productName: product.name(),
        quantity: dto.quantity,
        unitPrice: product.priceFor(customer),
        subtotal: dto.quantity * product.priceFor(customer)
      )
      
      lines.add(line)
    
    return lines
  
  method applyTaxRules(order, jurisdiction)
    taxRate = taxService.getRateFor(jurisdiction)
    taxAmount = order.subtotal() * taxRate
    order.setTax(taxAmount)
  
  method applyDiscount(order, loyaltyTier)
    discountRate = loyaltyService.discountRateFor(loyaltyTier)
    if discountRate > 0
      discountAmount = order.subtotal() * discountRate
      order.setDiscount(discountAmount)
```

### Abstract Factory Pattern (Multi-Aggregate)

```pseudocode
interface OrderManagementFactory
  method createOrder(customerId, lineItems) -> Order
  method createInvoice(order) -> Invoice
  method createShipment(order) -> Shipment

class EcommerceOrderFactory implements OrderManagementFactory
  method createOrder(customerId, lineItems) -> Order
    // ... validation and creation logic
    return order
  
  method createInvoice(order) -> Invoice
    invoice = new Invoice(
      id: InvoiceId.generate(),
      orderId: order.id(),
      customerId: order.customerId(),
      issueDate: now(),
      dueDate: now() + 30.days,
      items: mapOrderLinesToInvoiceItems(order.lines()),
      subtotal: order.subtotal(),
      tax: order.tax(),
      total: order.total(),
      status: InvoiceStatus.DRAFT
    )
    return invoice
  
  method createShipment(order) -> Shipment
    shipment = new Shipment(
      id: ShipmentId.generate(),
      orderId: order.id(),
      customerId: order.customerId(),
      shippingAddress: order.shippingAddress(),
      carrier: selectOptimalCarrier(order),
      trackingNumber: null,  // Will be assigned by carrier
      status: ShipmentStatus.PENDING,
      items: mapOrderLinesToShipmentItems(order.lines()),
      estimatedDelivery: calculateEstimatedDelivery(order)
    )
    return shipment
```

---

## Reconstitution from Persistence

Factories also handle **rehydration** of aggregates from the database. This differs from creation in that invariants are already satisfied (they were enforced when first created).

### Reconstitution Factory Method

```pseudocode
class OrderFactory
  method reconstructFromDatabase(dbRow) -> Order
    // Minimal validation—trust the database has valid state
    order = new Order(
      id: OrderId.fromString(dbRow.id),
      customerId: CustomerId.fromString(dbRow.customer_id),
      status: OrderStatus.fromString(dbRow.status),
      createdAt: dbRow.created_at,
      lines: reconstructOrderLines(dbRow.id),
      subtotal: Money.fromCents(dbRow.subtotal_cents),
      tax: Money.fromCents(dbRow.tax_cents),
      discount: Money.fromCents(dbRow.discount_cents),
      total: Money.fromCents(dbRow.total_cents),
      shippingAddress: reconstructAddress(dbRow.shipping_address_json),
      version: dbRow.version  // For optimistic locking
    )
    
    // Skip expensive validations since this came from trusted storage
    return order
  
  method reconstructOrderLines(orderId) -> List[OrderLine]
    rows = database.query(
      "SELECT * FROM order_lines WHERE order_id = ?",
      [orderId.value()]
    )
    
    lines = []
    for each row in rows
      line = new OrderLine(
        productId: ProductId.fromString(row.product_id),
        productName: row.product_name,
        quantity: row.quantity,
        unitPrice: Money.fromCents(row.unit_price_cents),
        subtotal: Money.fromCents(row.subtotal_cents)
      )
      lines.add(line)
    
    return lines
```

### Key Difference: Creation vs Reconstitution

| Aspect | Creation Factory | Reconstitution Factory |
|--------|------------------|------------------------|
| **Purpose** | Build new aggregate from user input | Rebuild aggregate from persistence |
| **Validation** | Full validation, enforce all invariants | Light validation, trust database |
| **Performance** | Slower (business logic checks) | Faster (minimal checks) |
| **Preconditions** | May need to load related entities | All needed data in database |
| **Error Handling** | Reject invalid input | Log/alert if corruption found |
| **Example** | `createFromCustomer()` | `reconstructFromDatabase()` |

---

## Factory vs Constructor Comparison

| Aspect | Constructor | Factory |
|--------|-------------|---------|
| **Complexity** | ✓ Simple, straightforward | ✓ Encapsulates complex logic |
| **Invariant Enforcement** | ✗ Relies on client discipline | ✓ Guarantees invariants |
| **Flexibility** | ✗ One way to create | ✓ Multiple creation strategies |
| **Encapsulation** | ✗ Exposes all aggregate fields | ✓ Hides internal structure |
| **Business Logic** | ✗ None in constructor | ✓ Business rules applied |
| **Testability** | ✓ Direct | ✓ Mock/test factory implementations |
| **Validation** | ✗ Limited (null checks) | ✓ Domain-specific validation |
| **Dependencies** | ✗ Passed to constructor | ✓ Injected into factory |
| **Entity Relationships** | ✗ Must handle manually | ✓ Factory loads related entities |
| **Use Case** | Value objects, simple entities | Complex aggregates |

---

## Summary Table

| Pattern | Purpose | Best For | Example |
|---------|---------|----------|---------|
| **Simple Constructor** | Direct initialization | Simple value objects, entities without invariants | `new Money(100, "USD")` |
| **Factory Method** | Multiple strategies for one aggregate | Creating order from different sources | `OrderFactory.createFromCustomer()`, `.createFromQuote()` |
| **Abstract Factory** | Creating families of related objects | Multi-aggregate creation with consistency | `EcommerceFactory.createOrder()`, `.createInvoice()`, `.createShipment()` |
| **Reconstitution** | Rebuilding aggregates from persistence | Rehydrating from database | `OrderFactory.reconstructFromDatabase()` |
| **Builder** (when combined with Factory) | Gradual construction with validation steps | Complex objects with many optional fields | `new OrderBuilder().withCustomer(...).withItems(...).build()` |

---

## Key Takeaways

1. **Factories encode creation knowledge**: Move the "how" of creation into a dedicated, reusable place
2. **Enforce invariants early**: Guarantees every aggregate is valid immediately
3. **Multiple strategies**: Factory methods allow different creation paths for the same aggregate
4. **Encapsulate structure**: Clients don't need to know aggregate internal details
5. **Separate reconstitution**: Different logic for creating new vs. rebuilding from persistence
6. **Inject dependencies**: Factories receive repositories, services, and other dependencies
7. **Stateless**: Factories should be stateless services, not state-holding objects

---

*Based on concepts from "Domain-Driven Design" by Eric Evans.*
