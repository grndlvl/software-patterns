# Anti-Corruption Layer (ACL)

## Definition

An **Anti-Corruption Layer (ACL)** is a strategic Domain-Driven Design pattern that acts as a protective boundary between your domain model and external systems (legacy applications, third-party services, different bounded contexts). It translates external representations into your domain language, preventing external models from corrupting your bounded context.

## Purpose

The Anti-Corruption Layer solves a fundamental problem in integration: **how do you integrate with external systems while keeping your domain model clean and uncompromised?**

### Core Objectives

| Objective | Benefit |
|-----------|---------|
| **Protect Domain Purity** | Prevent external system models from leaking into your core domain logic |
| **Isolate Integration Complexity** | Contain all translation and adaptation logic in one place |
| **Maintain Linguistic Consistency** | Speak your own domain language internally, regardless of external systems |
| **Enable Independent Evolution** | Change external integrations without modifying domain model |
| **Create Explicit Boundaries** | Make the contract between your system and external dependencies visible |

## When to Use

### Use Anti-Corruption Layer When:

1. **Integrating with Legacy Systems**
   - Old system uses different terminology and data structures
   - Cannot modify legacy system
   - Want to gradually migrate away from legacy

2. **Consuming Third-Party APIs**
   - External API schema doesn't match your domain model
   - API may change without warning
   - API language/concepts don't align with your domain

3. **Collaborating Across Bounded Contexts**
   - Partner team uses different ubiquitous language
   - Needs decoupling between independent teams
   - Different contexts evolve at different rates

4. **Wrapping Infrastructure Concerns**
   - Database schemas that don't match domain model
   - Message queue formats
   - Cache key structures

5. **Managing Technical Debt**
   - Gradual migration from one system to another
   - Phased integration rollout
   - Parallel running of old and new systems

### Don't Use ACL When:

- Integrating simple, well-aligned services
- Lightweight adapters are sufficient
- External system is fully under your control
- Performance is absolutely critical (extra translation layer adds overhead)

## Implementation Patterns

### 1. Facade Pattern

Creates a simplified, unified interface to a complex external subsystem.

```pseudocode
// External system (we don't control this)
class LegacyCustomerData
  properties:
    cust_id: String
    fname: String
    lname: String
    ph_num: String
    email_addr: String

// Our domain model
class Customer
  properties:
    id: CustomerId
    name: FullName
    contact: ContactInfo

// Anti-Corruption Layer: Facade
class LegacyCustomerFacade
  private legacyService: LegacyCustomerService

  function getCustomer(customerId: CustomerId) -> Customer
    legacyData = legacyService.fetchCustomer(customerId.value())
    return translateToCustomer(legacyData)

  private function translateToCustomer(legacyData: LegacyCustomerData) -> Customer
    id = CustomerId.create(legacyData.cust_id)
    fullName = FullName.create(
      legacyData.fname,
      legacyData.lname
    )
    contactInfo = ContactInfo.create(
      emailAddress: legacyData.email_addr,
      phoneNumber: legacyData.ph_num
    )
    return Customer(id, fullName, contactInfo)
```

**Characteristics:**
- One-way translation (legacy → domain)
- Hides complexity of external system
- Provides clean interface to internal code

### 2. Adapter Pattern

Converts the interface of a class into another interface that clients expect. Works bidirectionally.

```pseudocode
// External payment provider interface
class PaymentProviderAPI
  function chargeCard(
    cardToken: String,
    amountCents: Integer,
    merchantId: String
  ) -> PaymentResponse

// Our domain model
class Payment
  properties:
    orderId: OrderId
    amount: Money
    paymentMethod: PaymentMethod
    status: PaymentStatus

// Anti-Corruption Layer: Adapter
class PaymentProviderAdapter implements PaymentGateway
  private provider: PaymentProviderAPI
  private merchantConfig: MerchantConfiguration

  function processPayment(payment: Payment) -> PaymentResult
    // Adapt from domain model to provider API
    cardToken = extractCardToken(payment.paymentMethod)
    amountCents = convertToCents(payment.amount)
    merchantId = merchantConfig.getProviderId()

    // Call external service
    response = provider.chargeCard(
      cardToken,
      amountCents,
      merchantId
    )

    // Adapt response back to domain model
    return translateResponse(response, payment)

  private function translateResponse(
    response: PaymentResponse,
    payment: Payment
  ) -> PaymentResult
    
    status = mapProviderStatus(response.status)
    
    result = PaymentResult.create(
      paymentId: response.transaction_id,
      orderId: payment.orderId,
      amount: payment.amount,
      status: status
    )

    if response.has_error()
      result.setError(response.error_message)
    end if

    return result

  private function mapProviderStatus(providerStatus: String) -> PaymentStatus
    mapping = {
      "SUCCESS": PaymentStatus.Completed,
      "PENDING": PaymentStatus.Processing,
      "DECLINED": PaymentStatus.Failed,
      "ERROR": PaymentStatus.Error
    }
    return mapping[providerStatus]
```

**Characteristics:**
- Bidirectional translation
- Two-way interface (domain ↔ external)
- Maintains contracts on both sides

### 3. Translator (Mediator) Pattern

Explicitly handles translation logic between two models without being used as a service interface.

```pseudocode
// External warehouse system
class WarehouseInventory
  properties:
    product_code: String
    qty_on_hand: Integer
    qty_reserved: Integer
    last_updated: DateTime
    warehouse_location: String

// Our domain model
class Inventory
  properties:
    productId: ProductId
    availableQuantity: Quantity
    reservedQuantity: Quantity
    lastChecked: Timestamp
    location: WarehouseLocation

// Anti-Corruption Layer: Translator
class InventoryTranslator
  static function fromWarehouseData(
    warehouseData: WarehouseInventory
  ) -> Inventory
    
    productId = ProductId.create(warehouseData.product_code)
    
    available = Quantity.create(
      warehouseData.qty_on_hand - warehouseData.qty_reserved
    )
    reserved = Quantity.create(warehouseData.qty_reserved)
    
    lastChecked = Timestamp.from(warehouseData.last_updated)
    
    location = parseWarehouseLocation(warehouseData.warehouse_location)
    
    return Inventory(
      productId,
      available,
      reserved,
      lastChecked,
      location
    )

  static function toWarehouseData(
    inventory: Inventory
  ) -> WarehouseInventory
    
    return WarehouseInventory(
      product_code: inventory.productId.value(),
      qty_on_hand: inventory.availableQuantity.value() + 
                   inventory.reservedQuantity.value(),
      qty_reserved: inventory.reservedQuantity.value(),
      last_updated: inventory.lastChecked.toDateTime(),
      warehouse_location: formatLocation(inventory.location)
    )

  private static function parseWarehouseLocation(
    location: String
  ) -> WarehouseLocation
    // Parse "WAREHOUSE-A:SHELF-3:BIN-5" format
    parts = location.split(":")
    return WarehouseLocation.create(
      warehouse: parts[0],
      shelf: parts[1],
      bin: parts[2]
    )

  private static function formatLocation(
    location: WarehouseLocation
  ) -> String
    return location.warehouse + ":" + location.shelf + ":" + location.bin
```

**Characteristics:**
- Stateless translation functions
- Can be used standalone
- Easy to test and reuse
- Pure transformation logic

## Real-World Example: E-Commerce Integration

### Scenario

Your e-commerce system integrates with two different payment providers and a legacy billing system.

```pseudocode
// ============================================================================
// EXTERNAL SYSTEM 1: Payment Provider API (Stripe-like)
// ============================================================================

class StripeAPI
  function createCharge(
    amount: Integer,           // cents
    currency: String,          // "USD"
    source: String,            // token
    description: String
  ) -> StripeCharge

class StripeCharge
  properties:
    id: String
    amount: Integer
    currency: String
    status: String             // "succeeded", "failed", "pending"
    created: Long             // Unix timestamp

// ============================================================================
// EXTERNAL SYSTEM 2: Legacy Billing Service
// ============================================================================

class LegacyBillingServiceClient
  function recordCharge(
    cust_acct: String,
    amt: Decimal,
    desc: String,
    tx_date: String          // "YYYY-MM-DD HH:MM:SS"
  ) -> Integer                // returns: 1 = success, 0 = failure

// ============================================================================
// OUR DOMAIN MODEL
// ============================================================================

class Order
  properties:
    id: OrderId
    customerId: CustomerId
    totalAmount: Money        // value object with currency
    status: OrderStatus

class Payment
  properties:
    id: PaymentId
    orderId: OrderId
    amount: Money
    provider: PaymentProvider // enum: Stripe, PayPal, Square
    status: PaymentStatus     // Pending, Authorized, Captured, Failed
    transactionId: String
    timestamp: Timestamp

class PaymentProcessor
  // This should NOT import or know about StripeAPI or LegacyBillingService
  // It only knows about our domain abstractions

  function processPayment(order: Order, paymentMethod: PaymentMethod) -> Payment
    // Domain logic - no external concerns
    amount = order.totalAmount
    provider = paymentMethod.getProvider()
    
    payment = Payment.create(
      orderId: order.id,
      amount: amount,
      provider: provider
    )
    
    result = paymentGateway.authorize(payment)
    
    if result.successful()
      payment.markAuthorized(result.transactionId)
    else
      payment.markFailed(result.errorMessage)
    end if
    
    return payment

// ============================================================================
// ANTI-CORRUPTION LAYER: Payment Gateway Abstraction
// ============================================================================

interface PaymentGateway
  function authorize(payment: Payment) -> AuthorizationResult

// ============================================================================
// ACL: Stripe Adapter
// ============================================================================

class StripePaymentAdapter implements PaymentGateway
  private stripeAPI: StripeAPI
  private stripeConfig: StripeConfiguration

  function authorize(payment: Payment) -> AuthorizationResult
    // Translate domain model to Stripe API
    stripeCharge = stripeAPI.createCharge(
      amount: toCents(payment.amount),
      currency: payment.amount.currency.code,
      source: getTokenFromPaymentMethod(payment.paymentMethod),
      description: buildDescription(payment)
    )
    
    // Translate Stripe response back to domain
    return translateStripeResponse(stripeCharge, payment)

  private function translateStripeResponse(
    charge: StripeCharge,
    payment: Payment
  ) -> AuthorizationResult
    
    status = mapStripeStatus(charge.status)
    
    return AuthorizationResult.create(
      successful: status == PaymentStatus.Authorized,
      transactionId: charge.id,
      statusCode: charge.status,
      timestamp: Timestamp.fromUnix(charge.created)
    )

  private function mapStripeStatus(stripeStatus: String) -> PaymentStatus
    mapping = {
      "succeeded": PaymentStatus.Authorized,
      "failed": PaymentStatus.Failed,
      "pending": PaymentStatus.Pending
    }
    return mapping[stripeStatus] || PaymentStatus.Error

  private function toCents(money: Money) -> Integer
    return money.amount * 100

  private function buildDescription(payment: Payment) -> String
    return "Order " + payment.orderId.value()

// ============================================================================
// ACL: Legacy Billing System Adapter
// ============================================================================

class LegacyBillingAdapter
  private legacyClient: LegacyBillingServiceClient
  private accountMapper: LegacyAccountMapper

  function recordPayment(payment: Payment) -> BillingResult
    // Translate to legacy system format
    legacyResult = legacyClient.recordCharge(
      cust_acct: accountMapper.mapToLegacyAccount(payment.customerId),
      amt: toLegacyDecimal(payment.amount),
      desc: payment.transactionId,
      tx_date: formatDateForLegacy(payment.timestamp)
    )
    
    // Translate response back to our domain
    return BillingResult.create(
      success: legacyResult == 1,
      recorded: payment.timestamp
    )

  private function toLegacyDecimal(money: Money) -> Decimal
    return Decimal.from(money.amount)

  private function formatDateForLegacy(timestamp: Timestamp) -> String
    return timestamp.format("YYYY-MM-DD HH:MM:SS")

// ============================================================================
// DEPENDENCY INJECTION & COMPOSITION
// ============================================================================

// During application bootstrap
function setupPaymentProcessing()
  stripeConfig = loadStripeConfiguration()
  stripeAPI = StripeAPI.configure(stripeConfig)
  stripeAdapter = StripePaymentAdapter(stripeAPI, stripeConfig)
  
  legacyClient = LegacyBillingServiceClient(legacyConfig)
  legacyAdapter = LegacyBillingAdapter(legacyClient)
  
  // PaymentProcessor knows nothing about external systems
  processor = PaymentProcessor(
    paymentGateway: stripeAdapter,
    billingRecorder: legacyAdapter
  )
  
  return processor
```

## Comparison: With vs Without ACL

| Aspect | WITHOUT Anti-Corruption Layer | WITH Anti-Corruption Layer |
|--------|-------------------------------|---------------------------|
| **Domain Model** | Contaminated with external system concerns | Pure, focused on business logic |
| **External API Changes** | Force changes throughout domain code | Isolated to ACL adapter only |
| **Testing** | Hard to test domain logic without external system | Easy to mock/stub ACL layer |
| **Code Clarity** | Mixed concerns (business + integration) | Clear separation of concerns |
| **Team Communication** | Must understand external system details | Focus on domain language |
| **Maintainability** | High - multiple places to change | Low - centralized changes |
| **Performance** | Potentially faster (no translation) | Translation overhead per call |
| **Coupling** | Tight to external system | Loose coupling via abstraction |
| **Multiple Providers** | Complex conditional logic spread through code | Clean adapter pattern per provider |
| **Migration Path** | Difficult to swap systems | Easy - new adapter, same interface |

### Example: Processing Order Without ACL (Bad)

```pseudocode
class OrderProcessor
  function processOrder(rawOrderData: StripeOrderFormat) -> void
    // Domain logic mixed with external format handling
    customerId = rawOrderData["cust_id"]
    amount = rawOrderData["amt"] / 100  // What? Why dividing by 100?
    
    if rawOrderData["status"] == "succeeded"
      // Mark as completed
    else if rawOrderData["status"] == "failed"
      // Mark as failed
    end if

    // Later: need PayPal integration
    if source == "paypal"
      customerId = rawOrderData["customer_id"]  // Different field name!
      amount = rawOrderData["amount"]            // Already in dollars
      if rawOrderData["state"] == "created"      // Different status name!
        // mark as completed
      end if
    end if
```

**Problems:**
- Domain logic is entangled with data translation
- Each new payment provider requires changes to OrderProcessor
- Conditional logic explodes with multiple providers
- Hard to understand business intent

### Example: Processing Order With ACL (Good)

```pseudocode
class OrderProcessor
  private paymentGateway: PaymentGateway

  function processOrder(order: Order) -> void
    // Pure domain logic
    payment = Payment.create(orderId: order.id, amount: order.total)
    
    result = paymentGateway.authorize(payment)
    
    if result.successful()
      order.markPaid(result.transactionId)
    else
      order.markPaymentFailed(result.errorMessage)
    end if

// Adapter handles all translation
class StripeAdapter implements PaymentGateway
  function authorize(payment: Payment) -> AuthorizationResult
    stripeData = stripeAPI.createCharge(
      amount: payment.amount.toCents(),
      ...
    )
    return translate(stripeData)

class PayPalAdapter implements PaymentGateway
  function authorize(payment: Payment) -> AuthorizationResult
    paypalData = paypalAPI.executePayment(
      amount: payment.amount.toDecimal(),
      ...
    )
    return translate(paypalData)
```

**Benefits:**
- OrderProcessor contains only business logic
- Each provider gets its own adapter
- Easy to add new providers (just implement PaymentGateway)
- Clear, testable code

## Implementation Checklist

- [ ] **Identify External Dependencies** - List all external systems being integrated
- [ ] **Define Domain Model** - Create pure domain model without external concerns
- [ ] **Create Abstraction Layer** - Define interfaces representing external services
- [ ] **Implement Adapters** - Create adapter for each external system
- [ ] **Translation Logic** - Implement bi-directional translation (if needed)
- [ ] **Error Mapping** - Map external errors to domain errors
- [ ] **Configuration** - Externalize ACL configuration
- [ ] **Testing** - Unit test adapters independently
- [ ] **Dependency Injection** - Wire adapters during bootstrap
- [ ] **Documentation** - Document translation contracts
- [ ] **Monitoring** - Log translation failures and external API calls

## Anti-Patterns to Avoid

| Anti-Pattern | Problem | Solution |
|--------------|---------|----------|
| **Leaky ACL** | Domain model directly imports external libraries | Make ACL the only place that knows about external system |
| **God Adapter** | One massive adapter handling all translation | Create separate adapter per external system |
| **Bidirectional Without Necessity** | Translating both directions when only one direction needed | Keep it unidirectional if possible (simpler) |
| **Missing Error Translation** | Throwing external API errors to domain code | Map all external errors to domain error types |
| **No Abstraction** | Using external library classes as domain models | Always translate to domain model first |
| **Over-Caching** | Caching translated objects for too long | Keep translations fresh, invalidate appropriately |

## Summary Table

| Pattern | Use Case | Bidirectional | Complexity | Best For |
|---------|----------|:-------------:|:----------:|----------|
| **Facade** | One-way consumption of complex external system | ❌ | Low | Reading data from legacy/third-party |
| **Adapter** | Two-way integration, implement interface | ✅ | Medium | Service integration, provider switching |
| **Translator** | Pure transformation logic, reusable | ✅ | Low | Stateless conversions, data mapping |

---

*Based on concepts from "Domain-Driven Design" by Eric Evans.*
