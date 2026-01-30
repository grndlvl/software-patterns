# Factory Method

## Intent

Define an interface for creating an object, but let subclasses decide which class to instantiate. Factory Method lets a class defer instantiation to subclasses, promoting loose coupling between the creator and the concrete products.

## Also Known As

- Virtual Constructor
- Factory
- Creation Method

## Motivation

Consider a framework for applications that can work with different types of documents. The framework defines an abstract Application class that handles standard operations like opening, closing, and saving documents. It also defines an abstract Document class. Both classes are abstract because the framework doesn't know what specific kinds of documents will be created—that depends on the particular application using the framework.

To create a document, the Application class must instantiate a Document, but it only knows when a document should be created, not what kind of document to create. This creates a dilemma: the framework must instantiate classes, but it only knows about abstract classes, which it cannot instantiate.

The Factory Method pattern solves this problem. It encapsulates the knowledge of which Document subclass to create and moves this knowledge out of the framework. Application subclasses redefine an abstract CreateDocument operation on Application to return the appropriate Document subclass. We call CreateDocument a "factory method" because it's responsible for "manufacturing" an object.

Once an Application subclass is instantiated, it can then instantiate application-specific Documents without knowing their class. For example, a DrawingApplication creates DrawingDocuments, and a SpreadsheetApplication creates SpreadsheetDocuments. Each Application subclass defines its factory method to return the correct document type.

## Applicability

Use the Factory Method pattern when:

- A class can't anticipate the class of objects it must create
- A class wants its subclasses to specify the objects it creates
- Classes delegate responsibility to one of several helper subclasses, and you want to localize the knowledge of which helper subclass is the delegate
- You want to provide users of your library or framework a way to extend its internal components
- You want to save system resources by reusing existing objects instead of creating new ones each time
- You need to decouple the code that creates objects from the code that uses them

## Structure

```
┌────────────────────────────────────────────────────────────────────────────┐
│                              CREATOR                                       │
│  Declares the factory method, which returns an object of type Product     │
├────────────────────────────────────────────────────────────────────────────┤
│ + factoryMethod(): Product        «abstract»                               │
│ + someOperation(): void                                                    │
│   {                                                                        │
│     product = factoryMethod()                                              │
│     // use product...                                                      │
│   }                                                                        │
└────────────────────────────────────────────────────────────────────────────┘
                                    ▲
                                    │ extends
        ┌───────────────────────────┼───────────────────────────┐
        │                           │                           │
┌───────┴───────────────┐   ┌───────┴───────────────┐   ┌───────┴───────────┐
│   ConcreteCreatorA    │   │   ConcreteCreatorB    │   │  ConcreteCreatorC │
├───────────────────────┤   ├───────────────────────┤   ├───────────────────┤
│ + factoryMethod():    │   │ + factoryMethod():    │   │ + factoryMethod(): │
│     Product           │   │     Product           │   │     Product       │
│   {                   │   │   {                   │   │   {               │
│     return new        │   │     return new        │   │     return new    │
│       ProductA()      │   │       ProductB()      │   │       ProductC()  │
│   }                   │   │   }                   │   │   }               │
└───────────────────────┘   └───────────────────────┘   └───────────────────┘
        │                           │                           │
        │ creates                   │ creates                   │ creates
        ▼                           ▼                           ▼
┌────────────────────────────────────────────────────────────────────────────┐
│                          «interface»                                       │
│                            PRODUCT                                         │
├────────────────────────────────────────────────────────────────────────────┤
│ + operation(): void                                                        │
└────────────────────────────────────────────────────────────────────────────┘
                    ▲                               ▲
                    │ implements                    │ implements
        ┌───────────┴───────────┐       ┌──────────┴────────────┐
        │                       │       │                       │
┌───────┴───────────┐   ┌───────┴───────────┐   ┌───────────────┴───┐
│     ProductA      │   │     ProductB      │   │     ProductC      │
├───────────────────┤   ├───────────────────┤   ├───────────────────┤
│ + operation()     │   │ + operation()     │   │ + operation()     │
└───────────────────┘   └───────────────────┘   └───────────────────┘


PARAMETERIZED FACTORY METHOD VARIANT:

┌────────────────────────────────────────────────────────────────────────────┐
│                              CREATOR                                       │
├────────────────────────────────────────────────────────────────────────────┤
│ + createProduct(type: ProductType): Product                                │
│   {                                                                        │
│     switch(type) {                                                         │
│       case TYPE_A: return new ProductA()                                   │
│       case TYPE_B: return new ProductB()                                   │
│       default: throw UnknownProductException                               │
│     }                                                                      │
│   }                                                                        │
└────────────────────────────────────────────────────────────────────────────┘
```

## Participants

- **Product** (Document, Notification)
  - Defines the interface of objects the factory method creates

- **ConcreteProduct** (DrawingDocument, PdfDocument, EmailNotification, SmsNotification)
  - Implements the Product interface

- **Creator** (Application, NotificationService)
  - Declares the factory method, which returns an object of type Product. Creator may also define a default implementation of the factory method that returns a default ConcreteProduct object
  - May call the factory method to create a Product object

- **ConcreteCreator** (DrawingApplication, EmailNotificationService)
  - Overrides the factory method to return an instance of a ConcreteProduct

## Collaborations

- Creator relies on its subclasses to define the factory method so that it returns an instance of the appropriate ConcreteProduct.

- The factory method eliminates the need to bind application-specific classes into the code. The code only deals with the Product interface; therefore it can work with any user-defined ConcreteProduct classes.

## Consequences

### Benefits

- **Eliminates tight coupling**: Factory methods eliminate the need to bind application-specific classes into your code. The code only deals with the Product interface; therefore it can work with any user-defined ConcreteProduct classes.

- **Single Responsibility Principle**: You can move the product creation code into one place in the program, making the code easier to support.

- **Open/Closed Principle**: You can introduce new types of products into the program without breaking existing client code.

- **Flexibility for subclasses**: Providing hooks for subclasses is a more flexible way to provide an extended version of an object than creating an object directly. Creating objects directly is inflexible because it commits the creator to a particular class.

- **Connects parallel class hierarchies**: Factory methods can connect parallel class hierarchies—hierarchies where one hierarchy creates objects for another.

- **Deferred instantiation**: A class may want to defer creating the objects it uses. This is useful when the cost of creating objects is high and they might not be needed.

### Liabilities

- **Complexity from subclassing**: A potential disadvantage of factory methods is that clients might have to subclass the Creator class just to create a particular ConcreteProduct object. Subclassing is fine when the client has to subclass the Creator class anyway, but otherwise the client now must deal with another point of evolution.

- **Class proliferation**: Each new product type requires both a new product class and a new creator class, which can lead to many small classes.

- **Parallel hierarchies**: You end up with parallel hierarchies of creators and products, which must be kept in sync.

## Implementation

Consider the following implementation issues:

### 1. Two Major Varieties

The two main variations of the Factory Method pattern differ based on whether the Creator class is abstract or concrete:

```pseudocode
// Variation 1: Abstract Creator - no default implementation
abstract class Creator {
    // Subclasses MUST override
    abstract function createProduct(): Product

    function doSomething() {
        product = createProduct()
        product.operation()
    }
}

// Variation 2: Concrete Creator - provides default
class Creator {
    // Subclasses MAY override
    function createProduct(): Product {
        return new DefaultProduct()
    }

    function doSomething() {
        product = createProduct()
        product.operation()
    }
}
```

### 2. Parameterized Factory Methods

Another variation lets the factory method create multiple kinds of products. The factory method takes a parameter that identifies the kind of object to create:

```pseudocode
class NotificationFactory {
    function createNotification(channel: NotificationChannel): Notification {
        switch (channel) {
            case EMAIL:
                return new EmailNotification()
            case SMS:
                return new SmsNotification()
            case PUSH:
                return new PushNotification()
            case SLACK:
                return new SlackNotification()
            default:
                throw new UnknownChannelException(channel)
        }
    }
}

enum NotificationChannel {
    EMAIL, SMS, PUSH, SLACK
}
```

### 3. Using Generics/Templates

Factory methods can be made more flexible with generics:

```pseudocode
interface Creator<T extends Product> {
    function create(): T
}

class EmailNotificationCreator implements Creator<EmailNotification> {
    function create(): EmailNotification {
        return new EmailNotification()
    }
}

class SmsNotificationCreator implements Creator<SmsNotification> {
    function create(): SmsNotification {
        return new SmsNotification()
    }
}
```

### 4. Registry-Based Factory

For maximum flexibility, use a registry that maps identifiers to creation functions:

```pseudocode
class ProductRegistry {
    private static creators: Map<String, Function<Product>> = {}

    static function register(type: String, creator: Function<Product>) {
        creators.put(type, creator)
    }

    static function create(type: String): Product {
        creator = creators.get(type)
        if (creator == null) {
            throw new UnknownProductException(type)
        }
        return creator()
    }
}

// Registration (often done at startup)
ProductRegistry.register("email", () => new EmailNotification())
ProductRegistry.register("sms", () => new SmsNotification())
ProductRegistry.register("push", () => new PushNotification())

// Later usage
notification = ProductRegistry.create("email")
```

### 5. Lazy Initialization with Factory Methods

Factory methods are often used to implement lazy initialization:

```pseudocode
class ExpensiveResource {
    private static instance: ExpensiveResource = null

    static function getInstance(): ExpensiveResource {
        if (instance == null) {
            instance = new ExpensiveResource()
        }
        return instance
    }

    private constructor() {
        // Expensive initialization
    }
}
```

### 6. Factory Method with Dependency Injection

```pseudocode
interface NotificationCreator {
    function create(config: NotificationConfig): Notification
}

class EmailNotificationCreator implements NotificationCreator {
    private smtpClient: SmtpClient
    private templateEngine: TemplateEngine

    constructor(smtpClient: SmtpClient, templateEngine: TemplateEngine) {
        this.smtpClient = smtpClient
        this.templateEngine = templateEngine
    }

    function create(config: NotificationConfig): Notification {
        notification = new EmailNotification(smtpClient, templateEngine)
        notification.configure(config)
        return notification
    }
}

// DI Container configuration
container.register(NotificationCreator, EmailNotificationCreator)
container.inject(SmtpClient, new SmtpClient(settings))
container.inject(TemplateEngine, new TemplateEngine())
```

### 7. Avoiding String/Magic Values

Use enums or type-safe identifiers instead of strings:

```pseudocode
// Bad: Magic strings
notification = factory.create("email")

// Good: Type-safe enum
notification = factory.create(NotificationType.EMAIL)

// Better: Dedicated creator classes
creator = new EmailNotificationCreator()
notification = creator.create()
```

## Example

Here's a complete example of the Factory Method pattern applied to a payment processing system:

```pseudocode
// ============================================================
// PRODUCT INTERFACE
// ============================================================

interface PaymentProcessor {
    function processPayment(amount: Decimal, currency: String): PaymentResult
    function refund(transactionId: String, amount: Decimal): RefundResult
    function getTransactionStatus(transactionId: String): TransactionStatus
    function supportsRecurring(): Boolean
    function getSupportedCurrencies(): List<String>
}

class PaymentResult {
    success: Boolean
    transactionId: String
    errorMessage: String
    processorResponse: Map<String, Object>

    static function successful(transactionId: String): PaymentResult {
        result = new PaymentResult()
        result.success = true
        result.transactionId = transactionId
        return result
    }

    static function failed(errorMessage: String): PaymentResult {
        result = new PaymentResult()
        result.success = false
        result.errorMessage = errorMessage
        return result
    }
}

class RefundResult {
    success: Boolean
    refundId: String
    errorMessage: String
}

enum TransactionStatus {
    PENDING, COMPLETED, FAILED, REFUNDED, DISPUTED
}

// ============================================================
// CONCRETE PRODUCTS
// ============================================================

class StripeProcessor implements PaymentProcessor {
    private apiKey: String
    private webhookSecret: String

    constructor(apiKey: String, webhookSecret: String) {
        this.apiKey = apiKey
        this.webhookSecret = webhookSecret
    }

    function processPayment(amount: Decimal, currency: String): PaymentResult {
        try {
            // Stripe API call
            response = StripeApi.charges.create({
                "amount": convertToCents(amount),
                "currency": currency.toLowerCase(),
                "source": "tok_visa",  // In real code, this comes from client
                "api_key": apiKey
            })

            return PaymentResult.successful(response.id)
        } catch (StripeException e) {
            return PaymentResult.failed(e.getMessage())
        }
    }

    function refund(transactionId: String, amount: Decimal): RefundResult {
        response = StripeApi.refunds.create({
            "charge": transactionId,
            "amount": convertToCents(amount),
            "api_key": apiKey
        })

        result = new RefundResult()
        result.success = true
        result.refundId = response.id
        return result
    }

    function getTransactionStatus(transactionId: String): TransactionStatus {
        charge = StripeApi.charges.retrieve(transactionId, apiKey)
        if (charge.refunded) return TransactionStatus.REFUNDED
        if (charge.disputed) return TransactionStatus.DISPUTED
        if (charge.paid) return TransactionStatus.COMPLETED
        return TransactionStatus.PENDING
    }

    function supportsRecurring(): Boolean {
        return true
    }

    function getSupportedCurrencies(): List<String> {
        return ["USD", "EUR", "GBP", "CAD", "AUD", "JPY", ...]
    }

    private function convertToCents(amount: Decimal): Integer {
        return (amount * 100).toInteger()
    }
}

class PayPalProcessor implements PaymentProcessor {
    private clientId: String
    private clientSecret: String
    private sandboxMode: Boolean

    constructor(clientId: String, clientSecret: String, sandboxMode: Boolean) {
        this.clientId = clientId
        this.clientSecret = clientSecret
        this.sandboxMode = sandboxMode
    }

    function processPayment(amount: Decimal, currency: String): PaymentResult {
        try {
            accessToken = getAccessToken()

            response = PayPalApi.payments.create({
                "intent": "sale",
                "payer": {"payment_method": "paypal"},
                "transactions": [{
                    "amount": {
                        "total": amount.toString(),
                        "currency": currency
                    }
                }],
                "access_token": accessToken
            })

            return PaymentResult.successful(response.id)
        } catch (PayPalException e) {
            return PaymentResult.failed(e.getMessage())
        }
    }

    function refund(transactionId: String, amount: Decimal): RefundResult {
        accessToken = getAccessToken()
        saleId = getSaleIdFromPayment(transactionId)

        response = PayPalApi.sales.refund(saleId, {
            "amount": {
                "total": amount.toString(),
                "currency": getCurrencyFromTransaction(transactionId)
            },
            "access_token": accessToken
        })

        result = new RefundResult()
        result.success = true
        result.refundId = response.id
        return result
    }

    function getTransactionStatus(transactionId: String): TransactionStatus {
        accessToken = getAccessToken()
        payment = PayPalApi.payments.get(transactionId, accessToken)

        switch (payment.state) {
            case "approved": return TransactionStatus.COMPLETED
            case "created": return TransactionStatus.PENDING
            case "failed": return TransactionStatus.FAILED
            default: return TransactionStatus.PENDING
        }
    }

    function supportsRecurring(): Boolean {
        return true
    }

    function getSupportedCurrencies(): List<String> {
        return ["USD", "EUR", "GBP", "CAD", "AUD", ...]
    }

    private function getAccessToken(): String {
        // OAuth2 token retrieval
        baseUrl = sandboxMode
            ? "https://api.sandbox.paypal.com"
            : "https://api.paypal.com"
        // ... token retrieval logic
    }
}

class SquareProcessor implements PaymentProcessor {
    private accessToken: String
    private locationId: String

    constructor(accessToken: String, locationId: String) {
        this.accessToken = accessToken
        this.locationId = locationId
    }

    function processPayment(amount: Decimal, currency: String): PaymentResult {
        try {
            response = SquareApi.payments.create({
                "source_id": "cnon:card-nonce-ok",  // From client
                "idempotency_key": UUID.randomUUID().toString(),
                "amount_money": {
                    "amount": convertToCents(amount),
                    "currency": currency
                },
                "location_id": locationId,
                "access_token": accessToken
            })

            return PaymentResult.successful(response.payment.id)
        } catch (SquareException e) {
            return PaymentResult.failed(e.getMessage())
        }
    }

    function refund(transactionId: String, amount: Decimal): RefundResult {
        response = SquareApi.refunds.create({
            "payment_id": transactionId,
            "idempotency_key": UUID.randomUUID().toString(),
            "amount_money": {
                "amount": convertToCents(amount),
                "currency": getCurrencyFromPayment(transactionId)
            },
            "access_token": accessToken
        })

        result = new RefundResult()
        result.success = true
        result.refundId = response.refund.id
        return result
    }

    function getTransactionStatus(transactionId: String): TransactionStatus {
        payment = SquareApi.payments.get(transactionId, accessToken)

        switch (payment.status) {
            case "COMPLETED": return TransactionStatus.COMPLETED
            case "PENDING": return TransactionStatus.PENDING
            case "FAILED": return TransactionStatus.FAILED
            default: return TransactionStatus.PENDING
        }
    }

    function supportsRecurring(): Boolean {
        return false  // Requires Square Subscriptions API
    }

    function getSupportedCurrencies(): List<String> {
        return ["USD", "CAD", "GBP", "AUD", "JPY"]
    }

    private function convertToCents(amount: Decimal): Integer {
        return (amount * 100).toInteger()
    }
}

class MockProcessor implements PaymentProcessor {
    private shouldSucceed: Boolean = true
    private delay: Integer = 100

    function setSuccessMode(succeed: Boolean) {
        this.shouldSucceed = succeed
    }

    function setDelay(milliseconds: Integer) {
        this.delay = milliseconds
    }

    function processPayment(amount: Decimal, currency: String): PaymentResult {
        Thread.sleep(delay)

        if (shouldSucceed) {
            return PaymentResult.successful("mock_txn_" + UUID.randomUUID())
        } else {
            return PaymentResult.failed("Mock payment failure")
        }
    }

    function refund(transactionId: String, amount: Decimal): RefundResult {
        result = new RefundResult()
        result.success = shouldSucceed
        result.refundId = "mock_refund_" + UUID.randomUUID()
        return result
    }

    function getTransactionStatus(transactionId: String): TransactionStatus {
        return TransactionStatus.COMPLETED
    }

    function supportsRecurring(): Boolean {
        return true
    }

    function getSupportedCurrencies(): List<String> {
        return ["USD", "EUR", "GBP", "TEST"]
    }
}

// ============================================================
// CREATOR (FACTORY)
// ============================================================

abstract class PaymentProcessorFactory {
    // Template method that uses the factory method
    function getProcessor(): PaymentProcessor {
        processor = createProcessor()
        validateProcessor(processor)
        return processor
    }

    // Factory method - subclasses must implement
    protected abstract function createProcessor(): PaymentProcessor

    // Hook method - can be overridden
    protected function validateProcessor(processor: PaymentProcessor) {
        if (processor.getSupportedCurrencies().isEmpty()) {
            throw new InvalidProcessorException("Processor must support at least one currency")
        }
    }
}

// ============================================================
// CONCRETE CREATORS
// ============================================================

class StripeProcessorFactory extends PaymentProcessorFactory {
    private config: StripeConfiguration

    constructor(config: StripeConfiguration) {
        this.config = config
    }

    protected function createProcessor(): PaymentProcessor {
        return new StripeProcessor(
            config.getApiKey(),
            config.getWebhookSecret()
        )
    }
}

class PayPalProcessorFactory extends PaymentProcessorFactory {
    private config: PayPalConfiguration

    constructor(config: PayPalConfiguration) {
        this.config = config
    }

    protected function createProcessor(): PaymentProcessor {
        return new PayPalProcessor(
            config.getClientId(),
            config.getClientSecret(),
            config.isSandboxMode()
        )
    }
}

class SquareProcessorFactory extends PaymentProcessorFactory {
    private config: SquareConfiguration

    constructor(config: SquareConfiguration) {
        this.config = config
    }

    protected function createProcessor(): PaymentProcessor {
        return new SquareProcessor(
            config.getAccessToken(),
            config.getLocationId()
        )
    }
}

class MockProcessorFactory extends PaymentProcessorFactory {
    private successMode: Boolean = true

    function setSuccessMode(succeed: Boolean) {
        this.successMode = succeed
    }

    protected function createProcessor(): PaymentProcessor {
        processor = new MockProcessor()
        processor.setSuccessMode(successMode)
        return processor
    }
}

// ============================================================
// PARAMETERIZED FACTORY (Alternative approach)
// ============================================================

class PaymentProcessorFactoryRegistry {
    private static factories: Map<PaymentProvider, PaymentProcessorFactory> = {}

    static function register(provider: PaymentProvider, factory: PaymentProcessorFactory) {
        factories.put(provider, factory)
    }

    static function getFactory(provider: PaymentProvider): PaymentProcessorFactory {
        factory = factories.get(provider)
        if (factory == null) {
            throw new UnsupportedProviderException(
                "No factory registered for provider: " + provider
            )
        }
        return factory
    }

    static function createProcessor(provider: PaymentProvider): PaymentProcessor {
        return getFactory(provider).getProcessor()
    }
}

enum PaymentProvider {
    STRIPE, PAYPAL, SQUARE, MOCK
}

// ============================================================
// CLIENT CODE
// ============================================================

class CheckoutService {
    private processorFactory: PaymentProcessorFactory

    constructor(factory: PaymentProcessorFactory) {
        this.processorFactory = factory
    }

    function processOrder(order: Order, paymentDetails: PaymentDetails): OrderResult {
        processor = processorFactory.getProcessor()

        // Validate currency support
        if (!processor.getSupportedCurrencies().contains(order.getCurrency())) {
            throw new UnsupportedCurrencyException(order.getCurrency())
        }

        // Process payment
        paymentResult = processor.processPayment(
            order.getTotal(),
            order.getCurrency()
        )

        if (!paymentResult.success) {
            return OrderResult.failed(paymentResult.errorMessage)
        }

        // Save transaction
        transaction = new Transaction()
        transaction.orderId = order.getId()
        transaction.processorTransactionId = paymentResult.transactionId
        transaction.amount = order.getTotal()
        transaction.currency = order.getCurrency()
        transactionRepository.save(transaction)

        return OrderResult.successful(order.getId(), paymentResult.transactionId)
    }

    function refundOrder(orderId: String, amount: Decimal): RefundResult {
        transaction = transactionRepository.findByOrderId(orderId)
        processor = processorFactory.getProcessor()

        return processor.refund(transaction.processorTransactionId, amount)
    }
}

// ============================================================
// USAGE EXAMPLES
// ============================================================

function main() {
    // Load configuration
    config = Configuration.load()

    // Create appropriate factory based on environment
    factory: PaymentProcessorFactory

    if (config.getEnvironment() == "production") {
        providerConfig = config.getPaymentProvider()

        switch (providerConfig.getType()) {
            case "stripe":
                factory = new StripeProcessorFactory(providerConfig.asStripe())
                break
            case "paypal":
                factory = new PayPalProcessorFactory(providerConfig.asPayPal())
                break
            case "square":
                factory = new SquareProcessorFactory(providerConfig.asSquare())
                break
            default:
                throw new UnknownProviderException(providerConfig.getType())
        }
    } else {
        // Use mock for development/testing
        factory = new MockProcessorFactory()
    }

    // Create checkout service with the factory
    checkoutService = new CheckoutService(factory)

    // Process an order
    order = new Order()
    order.addItem("product-123", 2, 29.99)
    order.setCurrency("USD")

    result = checkoutService.processOrder(order, paymentDetails)

    if (result.isSuccessful()) {
        print("Order processed successfully: " + result.getTransactionId())
    } else {
        print("Order failed: " + result.getErrorMessage())
    }
}

// ============================================================
// TESTING WITH FACTORY METHOD
// ============================================================

class CheckoutServiceTest {
    function testSuccessfulPayment() {
        // Arrange
        mockFactory = new MockProcessorFactory()
        mockFactory.setSuccessMode(true)
        service = new CheckoutService(mockFactory)

        order = createTestOrder(100.00, "USD")

        // Act
        result = service.processOrder(order, testPaymentDetails)

        // Assert
        assert(result.isSuccessful())
        assertNotNull(result.getTransactionId())
    }

    function testFailedPayment() {
        // Arrange
        mockFactory = new MockProcessorFactory()
        mockFactory.setSuccessMode(false)
        service = new CheckoutService(mockFactory)

        order = createTestOrder(100.00, "USD")

        // Act
        result = service.processOrder(order, testPaymentDetails)

        // Assert
        assert(!result.isSuccessful())
        assertNotNull(result.getErrorMessage())
    }
}
```

## Known Uses

- **Java Collection Framework**: The `iterator()` method in Collection classes is a factory method that returns an Iterator appropriate for the collection type.

- **JDBC**: `DriverManager.getConnection()` returns a Connection implementation specific to the database driver.

- **Logging Frameworks**: `LoggerFactory.getLogger()` in SLF4J creates logger instances appropriate for the underlying logging implementation.

- **Spring Framework**: `BeanFactory.getBean()` creates and returns bean instances. The `@Bean` annotation marks factory methods.

- **JavaScript DOM**: `document.createElement()` is a factory method that creates different element types based on the tag name parameter.

- **Python's `__new__` method**: Acts as a factory method that can return different types or cached instances.

- **React.createElement()**: Creates React elements of different types based on the component type parameter.

- **Symfony/Laravel Service Containers**: Use factory methods to create and configure service instances.

- **java.util.Calendar.getInstance()**: Returns a Calendar implementation appropriate for the current locale.

- **java.text.NumberFormat.getInstance()**: Returns a number formatter appropriate for the current or specified locale.

## Related Patterns

- **Abstract Factory**: Abstract Factory classes are often implemented with factory methods. Abstract Factory creates families of objects; Factory Method creates one object.

- **Template Method**: Factory methods are often called from within template methods. The template method defines the algorithm, and factory methods create the objects the algorithm works with.

- **Prototype**: Factory Method doesn't require subclassing Creator but does require initializing the Product. Prototype doesn't require subclassing, but it does require an Initialize operation.

- **Singleton**: Factory methods can be used to implement Singleton by always returning the same instance.

- **Builder**: Builder focuses on constructing complex objects step by step. Factory Method creates objects in one step but allows subclasses to determine the type.

- **Dependency Injection**: Modern DI frameworks have largely replaced manual factory methods for object creation in many applications.

## When NOT to Use

- **Simple object creation**: If you're creating simple objects with straightforward constructors and no variation needed, direct instantiation is clearer.

- **Single implementation**: If there will only ever be one implementation of the product, the factory method abstraction adds unnecessary complexity.

- **No subclass customization needed**: If the creator class doesn't need to be extended and there's no need for different product types, skip the pattern.

- **Static utility classes**: For stateless utility operations that don't need instantiation at all.

- **When DI is available**: Modern dependency injection frameworks often provide a cleaner solution for managing object creation and dependencies.

**Simpler alternatives**:

- **Direct instantiation**: When the concrete class is known and won't change
  ```
  notification = new EmailNotification()
  ```

- **Simple Factory (not a GoF pattern)**: A static method in a class that creates objects based on parameters
  ```
  notification = NotificationFactory.create("email")
  ```

- **Constructor with dependency injection**: Pass the dependency directly
  ```
  service = new CheckoutService(new StripeProcessor(config))
  ```

- **Configuration-based selection**: Use configuration to select the implementation at runtime without subclassing
  ```
  processor = config.getProcessor()  // Returns configured implementation
  ```

**Signs you've over-engineered**:
- Your factory has only one concrete creator that's never extended
- You created a factory to instantiate a class with a simple constructor
- The "product" classes are trivial value objects
- You're never actually switching between different product implementations
- Your factory method just calls `new ConcreteProduct()` with no additional logic
- You have more factory infrastructure than actual product logic

---

*Based on concepts from "Design Patterns: Elements of Reusable Object-Oriented Software" by Gamma, Helm, Johnson, and Vlissides (Gang of Four), 1994.*
