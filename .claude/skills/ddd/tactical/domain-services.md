# Domain Services

## Definition

**Domain Services** are stateless operations that express important business logic that doesn't naturally belong to an Entity or Value Object. They encapsulate operations that span multiple Aggregates, coordinate between domain concepts, or represent core domain logic that isn't tied to a single domain object's lifecycle.

Unlike Entities (which have identity and lifecycle) and Value Objects (which represent descriptive characteristics), Domain Services represent **actions and processes** in the domain. They are typically stateless and operate on the domain concepts to coordinate their interaction.

A Domain Service should be named using domain language that describes what business process it performs.

## When to Use Domain Services

### 1. Operations Spanning Multiple Aggregates

When business logic involves coordinating between two or more Aggregates:

```pseudocode
# A money transfer involves two separate aggregates: source and destination accounts
# This coordination logic belongs in a Domain Service

class MoneyTransferService:
    function transfer(sourceAccount: Account, 
                     destinationAccount: Account, 
                     amount: Money):\
        # Logic that coordinates between two aggregates
        # Should NOT live in either Account entity
```

### 2. External System Interactions

When domain logic needs to interact with external systems or repositories:

```pseudocode
# Pricing logic that queries product databases or pricing engines
class PricingService:
    function calculatePrice(product: Product, 
                           customer: Customer): Money
        # May query external pricing rules, discounts, taxes
```

### 3. Complex Domain Algorithms

When complex domain-specific algorithms don't fit naturally in any single Entity:

```pseudocode
# Calculating shipping costs based on multiple factors
class ShippingCalculationService:
    function calculateShippingCost(order: Order, 
                                  destination: Address): Money
        # Complex algorithm involving weight, distance, carrier rates
```

### 4. Coordination of Specifications

When applying domain-driven specifications that cut across multiple aggregates:

```pseudocode
# Finding eligible products for a customer based on business rules
class EligibilityService:
    function findEligibleProducts(customer: Customer, 
                                 category: ProductCategory): List<Product>
        # Complex filtering based on customer status, promotions, availability
```

## Domain Service vs Application Service vs Infrastructure Service

**Critical distinction**: Domain Services contain **core business logic**, while Application Services coordinate use cases and Infrastructure Services handle technical concerns.

| Aspect | Domain Service | Application Service | Infrastructure Service |
|--------|---|---|---|
| **Purpose** | Encapsulate core business logic | Coordinate use cases and workflows | Handle technical/implementation concerns |
| **Domain Aware** | YES - knows business concepts | YES - coordinates domain objects | NO - implementation detail focused |
| **Stateless** | YES | YES (typically) | Often YES |
| **Contains Business Rules** | YES - core logic | NO - orchestrates | NO - technical logic |
| **Dependencies** | Domain objects (Entities, Aggregates) | Domain Services, Repositories, Application logic | External systems, databases, APIs |
| **Naming** | Business language (TransferService) | Use case language (TransferMoneyUseCase) | Technical language (SmtpEmailService) |
| **When to Use** | Operations on domain objects | Coordinating a user action | Talking to external systems |
| **Example** | PricingService, TransferService | ProcessOrderService, CreateUserUseCase | EmailSendingService, DatabaseQueryService |
| **Dependency Flow** | Domain Service ← Application Service | Application Service ← Domain Service, Repository | Infrastructure Service ← Application Service |
| **Reusability** | Across multiple use cases | Specific to one use case | Across entire application |

### Example: Fund Transfer

```pseudocode
# DOMAIN SERVICE - core business logic
class MoneyTransferService:
    function transfer(source: BankAccount, 
                     destination: BankAccount, 
                     amount: Money):\
        # This is DOMAIN LOGIC
        # It expresses the business rule: "money moves from one account to another"
        # It operates on domain objects (Accounts) with domain concepts (Money)
        
        source.withdraw(amount)  # Enforces account invariants
        destination.deposit(amount)  # Enforces account invariants
        
        raiseEvent(new MoneyTransferredEvent(
            sourceAccountId: source.accountId,
            destinationAccountId: destination.accountId,
            amount: amount
        ))


# APPLICATION SERVICE - use case coordination
class TransferMoneyUseCase:
    constructor(
        accountRepository: AccountRepository,
        transferService: MoneyTransferService,
        unitOfWork: UnitOfWork
    ):
        this.accountRepository = accountRepository
        this.transferService = transferService
        this.unitOfWork = unitOfWork
    
    function execute(command: TransferMoneyCommand):
        # 1. Get domain objects
        sourceAccount = this.accountRepository.findById(command.sourceAccountId)
        destinationAccount = this.accountRepository.findById(command.destinationAccountId)
        
        # 2. Call domain service with business logic
        this.transferService.transfer(
            sourceAccount,
            destinationAccount,
            command.amount
        )
        
        # 3. Persist changes
        this.accountRepository.save(sourceAccount)
        this.accountRepository.save(destinationAccount)
        this.unitOfWork.commit()


# INFRASTRUCTURE SERVICE - technical implementation
class SmtpEmailService:
    # This does NOT know about domain concepts
    # It's purely technical - sending emails via SMTP
    
    function sendEmail(to: String, subject: String, body: String):
        # Connect to SMTP server
        # Send message
        # Handle connection errors
        # This is INFRASTRUCTURE, not DOMAIN
```

## Characteristics of Domain Services

### 1. Stateless

Domain Services maintain no state. The same instance can safely handle multiple requests:

```pseudocode
class DiscountCalculationService:
    # NO instance variables
    # NO mutable state
    
    function calculateDiscount(order: Order): Money
        # Pure function behavior - no state
        # Always returns same result for same input
        return this.calculateBaseDiscount(order)
                .add(this.calculateLoyaltyDiscount(order))
                .add(this.calculateSeasonalDiscount(order))
```

### 2. Named After Domain Concepts

Use domain language, not technical language:

```pseudocode
# GOOD - expresses domain concept
class PricingService
class InventoryReservationService
class ShippingCalculationService

# BAD - technical language
class CalculatorHelper
class ProcessorUtils
class ServiceImpl

# GOOD - domain action
class MoneyTransferService
class InvoiceGenerationService

# BAD - vague language
class ManagementService
class HandlerService
```

### 3. Operates on Domain Objects

Domain Services work with Entities, Aggregates, and Value Objects - not on raw primitives:

```pseudocode
# GOOD - operates on domain objects
class OrderValidationService:
    function isValidForSubmission(order: Order): Boolean
        # Takes domain object, uses domain concepts
        return order.hasItems() AND order.totalAmount().isGreaterThan(Money.zero())


# BAD - operates on primitives
class OrderValidationService:
    function isValid(items: List<Item>, total: Decimal): Boolean
        # This doesn't feel like domain logic
        return items.length > 0 AND total > 0
```

### 4. Expresses Ubiquitous Language

The Domain Service name and methods should use terms from the Ubiquitous Language:

```pseudocode
# Example from ecommerce domain
class PricingService:
    # These methods use domain language
    function calculateFinalPrice(product: Product, 
                                customer: Customer): Money
    
    function applyPromotionalDiscount(basePrice: Money, 
                                      promotion: Promotion): Money
    
    function calculateTaxableAmount(order: Order): Money
```

## When NOT to Use Domain Services

### Don't Create a Domain Service If:

1. **Logic belongs in an Entity**
   ```pseudocode
   # WRONG - this is Entity behavior
   class PaymentService:
       function processPayment(payment: Payment):
           payment.markAsProcessed()
   
   # RIGHT - this belongs in the Entity
   class Payment:
       function processPayment():
           # ... validate and update state
           this.status = PaymentStatus.PROCESSED
   ```

2. **Logic belongs in a Value Object**
   ```pseudocode
   # WRONG - currency conversion is value behavior
   class CurrencyConversionService:
       function convert(amount: Money, targetCurrency: String): Money
   
   # RIGHT - if it's pure calculation/transformation
   class Money:
       function convertTo(targetCurrency: String, rate: Decimal): Money
           return new Money(this.amount * rate, targetCurrency)
   ```

3. **It's really an Application Service**
   ```pseudocode
   # WRONG - this is use case orchestration
   class CreateOrderService:  # (as Domain Service)
       function createOrder(dto: CreateOrderDTO):
           order = new Order(...)
           repository.save(order)  # Infrastructure concern
   
   # RIGHT - move to Application Service
   class CreateOrderUseCase:  # (Application Service)
       function execute(command: CreateOrderCommand):
           order = new Order(...)
           repository.save(order)
   ```

## Pseudocode Examples

### Example 1: TransferService

```pseudocode
# Domain Service for transferring money between accounts
class MoneyTransferService:
    # Stateless - no instance variables
    
    # Named after domain concept
    function transfer(fromAccount: BankAccount, 
                     toAccount: BankAccount, 
                     amount: Money):
        # Coordinates between two aggregates
        # Expresses core business logic: money moves atomically
        
        # Validate the transfer can occur
        if not fromAccount.canWithdraw(amount):
            throw new InsufficientFundsException(
                required: amount,
                available: fromAccount.getBalance()
            )
        
        if toAccount.isClosed():
            throw new AccountClosedException(toAccount.accountId)
        
        # Apply the transfer atomically
        fromAccount.withdraw(amount)
        toAccount.deposit(amount)
        
        # Raise domain event
        raiseEvent(new MoneyTransferredEvent(
            fromAccountId: fromAccount.accountId,
            toAccountId: toAccount.accountId,
            amount: amount,
            timestamp: Timestamp.now()
        ))
    
    # Helper method - still pure logic
    function validateTransfer(fromAccount: BankAccount, 
                             toAccount: BankAccount, 
                             amount: Money): ValidationResult
        errors = []
        
        if fromAccount.isNull():
            errors.add("Source account not found")
        
        if toAccount.isNull():
            errors.add("Destination account not found")
        
        if amount.isLessThanOrEqual(Money.zero(amount.currency)):
            errors.add("Transfer amount must be positive")
        
        if fromAccount != null AND not fromAccount.canWithdraw(amount):
            errors.add("Insufficient funds")
        
        if toAccount != null AND toAccount.isClosed():
            errors.add("Destination account is closed")
        
        return new ValidationResult(errors)
```

### Example 2: PricingService

```pseudocode
# Domain Service for calculating product prices with business rules
class PricingService:
    # Stateless - operates on domain objects passed to it
    
    function calculatePrice(product: Product, 
                           customer: Customer, 
                           quantity: Integer): Money
        # Core business logic: how to price a product for a customer
        
        basePrice = product.basePrice
        
        # Apply customer-specific pricing
        customerPrice = this.applyCustomerPricing(basePrice, customer)
        
        # Apply quantity discounts
        discountedPrice = this.applyQuantityDiscount(customerPrice, quantity)
        
        # Apply promotional discounts
        promotionalPrice = this.applyPromotionalDiscount(discountedPrice, product)
        
        return promotionalPrice
    
    function calculateOrderTotal(order: Order): Money
        # Aggregate calculation across multiple items
        total = Money.zero(order.currency)
        
        for item in order.getItems():
            itemPrice = this.calculatePrice(
                item.product,
                order.customer,
                item.quantity
            )
            lineTotal = itemPrice.multiply(item.quantity)
            total = total.add(lineTotal)
        
        # Apply order-level discounts
        total = this.applyOrderDiscount(total, order)
        
        return total
    
    private function applyCustomerPricing(basePrice: Money, 
                                         customer: Customer): Money
        if customer.isCorporate():
            # Corporate customers get 10% discount
            return basePrice.multiply(0.9)
        
        if customer.isPremiumMember():
            # Premium members get 5% discount
            return basePrice.multiply(0.95)
        
        # Regular customers pay full price
        return basePrice
    
    private function applyQuantityDiscount(price: Money, 
                                          quantity: Integer): Money
        if quantity >= 100:
            return price.multiply(0.85)  # 15% discount for bulk
        
        if quantity >= 50:
            return price.multiply(0.90)  # 10% discount
        
        if quantity >= 10:
            return price.multiply(0.95)  # 5% discount
        
        return price
    
    private function applyPromotionalDiscount(price: Money, 
                                             product: Product): Money
        activePromo = product.getActivePromotion()
        
        if activePromo == null:
            return price
        
        if activePromo.isPercentageDiscount():
            discountAmount = price.multiply(activePromo.discountPercentage / 100)
            return price.subtract(discountAmount)
        else:
            return price.subtract(activePromo.discountAmount)
    
    private function applyOrderDiscount(total: Money, 
                                       order: Order): Money
        # Loyalty-based discount at order level
        if order.customer.getLoyaltyPoints() > 1000:
            loyaltyDiscount = Money(10, total.currency)
            if total.isGreaterThan(loyaltyDiscount):
                return total.subtract(loyaltyDiscount)
        
        return total
```

### Example 3: InventoryReservationService

```pseudocode
# Domain Service for reserving inventory across multiple products
class InventoryReservationService:
    # No state - coordinates between inventory aggregates
    
    function reserveForOrder(order: Order): ReservationResult
        # Try to reserve all items in the order
        # If any item can't be reserved, release all previous reservations
        
        reservedItems = []
        
        try:
            for orderItem in order.getItems():
                product = orderItem.getProduct()
                quantity = orderItem.quantity
                
                if not product.inventory.canReserve(quantity):
                    throw new InsufficientInventoryException(
                        productId: product.productId,
                        requested: quantity,
                        available: product.inventory.availableQuantity()
                    )
                
                reservation = product.inventory.reserve(
                    quantity: quantity,
                    orderId: order.orderId,
                    reservationExpiresAt: Timestamp.now().plusHours(24)
                )
                
                reservedItems.add(reservation)
                raiseEvent(new InventoryReservedEvent(product.productId, quantity))
            
            return new ReservationResult(
                success: true,
                reservations: reservedItems
            )
        
        catch error:
            # Release all reservations made so far
            for reservation in reservedItems:
                reservation.release()
                raiseEvent(new ReservationReleasedEvent(reservation.productId))
            
            return new ReservationResult(
                success: false,
                error: error.message,
                reservations: []
            )
    
    function fulfillReservation(order: Order):
        # Convert reservations to actual inventory deductions
        
        for orderItem in order.getItems():
            product = orderItem.getProduct()
            quantity = orderItem.quantity
            
            reservation = product.inventory.findReservation(order.orderId)
            
            if reservation == null:
                throw new ReservationNotFoundException(order.orderId)
            
            if reservation.isExpired():
                throw new ExpiredReservationException(order.orderId)
            
            product.inventory.deduct(quantity)
            reservation.confirm()
            
            raiseEvent(new InventoryFulfilledEvent(
                productId: product.productId,
                quantity: quantity,
                orderId: order.orderId
            ))
    
    function releaseReservation(orderId: OrderId, product: Product):
        # Release a reservation (e.g., order cancelled)
        
        reservation = product.inventory.findReservation(orderId)
        
        if reservation != null:
            reservation.release()
            raiseEvent(new ReservationReleasedEvent(product.productId))
```

### Example 4: ShippingCalculationService

```pseudocode
# Domain Service for calculating shipping costs based on business rules
class ShippingCalculationService:
    
    function calculateShippingCost(order: Order, 
                                  destination: Address): Money
        # Core logic: how much does it cost to ship this order?
        
        baseRate = this.getBaseShippingRate(destination)
        weight = order.getTotalWeight()
        
        costByWeight = this.calculateWeightBasedCost(baseRate, weight)
        costByDistance = this.calculateDistanceBasedCost(baseRate, destination)
        
        totalCost = costByWeight.add(costByDistance)
        
        # Apply shipping discounts
        totalCost = this.applyShippingDiscount(totalCost, order)
        
        # Apply carrier surcharges
        totalCost = this.applyCarrierSurcharges(totalCost, destination)
        
        return totalCost
    
    private function getBaseShippingRate(destination: Address): Money
        # Different rates for different regions
        
        if destination.country == "US":
            return Money(5.00, "USD")
        
        if destination.country in ["CA", "MX"]:
            return Money(10.00, "USD")
        
        # International
        return Money(25.00, "USD")
    
    private function calculateWeightBasedCost(baseRate: Money, 
                                             weight: Weight): Money
        # Weight-based surcharge
        # $0.50 per pound over 5 pounds
        
        freeWeightThreshold = Weight(5, "lb")
        
        if weight.isLessThanOrEqual(freeWeightThreshold):
            return Money.zero(baseRate.currency)
        
        excessWeight = weight.subtract(freeWeightThreshold)
        excessPounds = excessWeight.toUnit("lb").value
        
        surcharge = Money(0.50, baseRate.currency).multiply(excessPounds)
        return surcharge
    
    private function calculateDistanceBasedCost(baseRate: Money, 
                                               destination: Address): Money
        # Distance-based surcharge
        
        distance = this.calculateDistance(destination)
        
        if distance < 100:
            multiplier = 1.0
        else if distance < 500:
            multiplier = 1.25
        else if distance < 1000:
            multiplier = 1.50
        else:
            multiplier = 2.0
        
        return baseRate.multiply(multiplier - 1.0)
    
    private function applyShippingDiscount(cost: Money, 
                                          order: Order): Money
        # Free shipping for orders over $100
        
        if order.subtotal.isGreaterThan(Money(100, order.currency)):
            return Money.zero(cost.currency)
        
        # Loyalty discount: loyal customers get 10% off shipping
        if order.customer.isLoyalty():
            discount = cost.multiply(0.10)
            return cost.subtract(discount)
        
        return cost
    
    private function applyCarrierSurcharges(cost: Money, 
                                           destination: Address): Money
        # Alaska and Hawaii surcharges
        
        if destination.state in ["AK", "HI"]:
            surcharge = cost.multiply(0.50)  # 50% surcharge
            return cost.add(surcharge)
        
        return cost
    
    private function calculateDistance(destination: Address): Integer
        # Calculate distance from distribution center
        # Returns distance in miles
        
        distributionCenter = this.getClosestDistributionCenter(destination)
        return this.getDistanceBetween(distributionCenter, destination)
```

## Domain Service Pattern: Complete Example

```pseudocode
# Complete example: subscription renewal service

class SubscriptionRenewalService:
    # Stateless - no instance variables
    # All dependencies injected or passed as parameters
    
    function renewSubscription(subscription: Subscription, 
                              paymentProcessor: PaymentProcessor): RenewalResult
        # Core business logic for subscription renewal
        
        # Validate subscription can be renewed
        if not subscription.isEligibleForRenewal():
            return new RenewalResult(
                success: false,
                reason: "Subscription not eligible for renewal"
            )
        
        # Calculate renewal amount
        renewalAmount = this.calculateRenewalAmount(subscription)
        
        # Attempt payment
        paymentResult = paymentProcessor.charge(
            customer: subscription.customer,
            amount: renewalAmount,
            description: "Subscription renewal for: " + subscription.planName
        )
        
        if not paymentResult.isSuccessful():
            raiseEvent(new SubscriptionRenewalFailedEvent(
                subscriptionId: subscription.subscriptionId,
                reason: paymentResult.failureReason
            ))
            
            return new RenewalResult(
                success: false,
                reason: paymentResult.failureReason
            )
        
        # Apply renewal to subscription
        subscription.renew(
            renewalDate: Date.now(),
            nextRenewalDate: this.calculateNextRenewalDate(subscription),
            amount: renewalAmount
        )
        
        raiseEvent(new SubscriptionRenewedEvent(
            subscriptionId: subscription.subscriptionId,
            amount: renewalAmount,
            nextRenewalDate: subscription.nextRenewalDate
        ))
        
        return new RenewalResult(
            success: true,
            renewalAmount: renewalAmount,
            nextRenewalDate: subscription.nextRenewalDate
        )
    
    function upgradeSubscription(subscription: Subscription, 
                                newPlan: Plan, 
                                paymentProcessor: PaymentProcessor): UpgradeResult
        # Core logic for upgrading subscriptions
        
        if not newPlan.tier > subscription.plan.tier:
            throw new DowngradeNotAllowedThroughUpgradeException()
        
        # Calculate prorated amount
        proratedAmount = this.calculateProratedAmount(
            subscription: subscription,
            newPlan: newPlan
        )
        
        if proratedAmount.isGreaterThan(Money.zero()):
            # Charge the difference
            paymentResult = paymentProcessor.charge(
                customer: subscription.customer,
                amount: proratedAmount,
                description: "Upgrade from " + subscription.plan.name + 
                           " to " + newPlan.name
            )
            
            if not paymentResult.isSuccessful():
                return new UpgradeResult(
                    success: false,
                    reason: paymentResult.failureReason
                )
        
        # Apply upgrade
        oldPlan = subscription.plan
        subscription.upgradeToPlan(newPlan)
        
        raiseEvent(new SubscriptionUpgradedEvent(
            subscriptionId: subscription.subscriptionId,
            fromPlan: oldPlan.name,
            toPlan: newPlan.name,
            proratedAmount: proratedAmount
        ))
        
        return new UpgradeResult(
            success: true,
            proratedAmount: proratedAmount,
            newPlan: newPlan
        )
    
    private function calculateRenewalAmount(subscription: Subscription): Money
        baseAmount = subscription.plan.monthlyPrice
        
        # Apply any active promotions
        if subscription.hasActivePromotionCode():
            discount = this.applyPromotionCode(baseAmount, subscription.promotionCode)
            return baseAmount.subtract(discount)
        
        return baseAmount
    
    private function calculateNextRenewalDate(subscription: Subscription): Date
        # Renewal happens 1 month from now
        return Date.now().plusMonths(1)
    
    private function calculateProratedAmount(subscription: Subscription, 
                                            newPlan: Plan): Money
        # Calculate the difference in monthly price
        oldMonthlyPrice = subscription.plan.monthlyPrice
        newMonthlyPrice = newPlan.monthlyPrice
        monthlyDifference = newMonthlyPrice.subtract(oldMonthlyPrice)
        
        # Pro-rate based on days remaining in billing period
        daysRemaining = subscription.getRenewalDate()
                          .differenceInDays(Date.now())
        daysInMonth = 30  # Simplified
        
        proratedAmount = monthlyDifference
            .multiply(daysRemaining / daysInMonth)
        
        # Return only positive amounts (customer pays extra)
        if proratedAmount.isLessThanOrEqual(Money.zero()):
            return Money.zero(proratedAmount.currency)
        
        return proratedAmount
    
    private function applyPromotionCode(amount: Money, 
                                       promotionCode: String): Money
        # Look up promotion and calculate discount
        # (In real code, this might query a repository)
        
        promotions = [
            ("SAVE10", amount.multiply(0.10)),
            ("SAVE20", amount.multiply(0.20)),
            ("SAVE50", amount.multiply(0.50))
        ]
        
        discount = promotions.find(p => p[0] == promotionCode)?[1]
        
        if discount != null:
            return discount
        
        return Money.zero(amount.currency)
```

## Summary Table

| Aspect | Details |
|--------|---------|
| **Purpose** | Express business logic that spans multiple aggregates or doesn't fit in an entity/value object |
| **Statefulness** | Stateless - maintains no instance variables |
| **Identity** | No identity - not persisted as domain objects |
| **Naming** | Domain-language verbs: TransferService, PricingService, InventoryReservationService |
| **Key Characteristic** | Coordinates between aggregates while expressing core business rules |
| **Operations** | Accept domain objects, perform business logic, return results or raise events |
| **Reusability** | Used across multiple application services and use cases |
| **Testing** | Tested with domain objects, not with mocks or DTOs |
| **Examples** | MoneyTransferService, PricingService, ShippingCalculationService, SubscriptionRenewalService |
| **When Multiple Aggregates Interact** | The coordinating logic typically lives here, not in either aggregate |
| **External Interactions** | May coordinate with repositories or other services, but encapsulates domain logic |
| **Distinction from Application Service** | Domain Services contain WHAT to do (business rules); Application Services coordinate HOW to do it (orchestration) |

---

*Based on concepts from "Domain-Driven Design" by Eric Evans.*
