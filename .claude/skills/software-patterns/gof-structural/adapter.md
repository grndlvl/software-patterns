# Adapter

## Intent

Convert the interface of a class into another interface that clients expect. Adapter lets classes work together that could not otherwise because of incompatible interfaces. It acts as a bridge between two incompatible interfaces by wrapping an existing class with a new interface.

## Also Known As

- Wrapper
- Translator

## Motivation

Consider a drawing application that allows users to manipulate graphical objects like lines, rectangles, and text. The application defines an abstract `Shape` class with operations like `boundingBox()` and `draw()`. For each kind of shape, there is a concrete subclass: `LineShape` for lines, `RectangleShape` for rectangles, and so on.

Now suppose you want to add a `TextShape` class that can display and edit text. However, implementing text editing from scratch is difficult, so you decide to reuse an existing `TextView` class from a GUI toolkit. Unfortunately, `TextView` was not designed with the `Shape` interface in mind. It has methods like `getExtent()` instead of `boundingBox()` and `render()` instead of `draw()`.

You could modify `TextView` to conform to the `Shape` interface, but this is not always possible. You may not have the source code, or modifying the class may break other parts of the system that depend on its original interface. Additionally, even if you could modify it, you should not; `TextView` is a reusable component that should remain independent of specific applications.

The Adapter pattern solves this problem by defining a `TextShape` class that adapts the `TextView` interface to the `Shape` interface. The adapter acts as a wrapper around `TextView`, translating calls from the `Shape` interface into calls that `TextView` understands. Clients call operations on the adapter, which in turn calls the appropriate operations on the adaptee.

## Applicability

Use the Adapter pattern when:

- You want to use an existing class, and its interface does not match the one you need.

- You want to create a reusable class that cooperates with unrelated or unforeseen classes, that is, classes that do not necessarily have compatible interfaces.

- You need to use several existing subclasses, but it is impractical to adapt their interface by subclassing every one. An object adapter can adapt the interface of its parent class.

- You want to integrate a third-party library or legacy code into your application without modifying the original source.

- You are working with multiple APIs that perform similar functions but have different interfaces, and you want to provide a unified interface.

- You need to convert data formats between systems (e.g., XML to JSON, different date formats).

## Structure

There are two types of Adapter patterns: Class Adapter (using inheritance) and Object Adapter (using composition).

### Object Adapter (Preferred)

```
                    ┌─────────────────────┐
                    │      <<interface>>  │
                    │        Target       │
                    ├─────────────────────┤
                    │ + request()         │
                    └─────────────────────┘
                              △
                              │ implements
                              │
┌─────────────┐      ┌─────────────────────┐      ┌─────────────────────┐
│   Client    │─────▶│      Adapter        │─────▶│      Adaptee        │
└─────────────┘      ├─────────────────────┤      ├─────────────────────┤
                     │ - adaptee: Adaptee  │      │ + specificRequest() │
                     ├─────────────────────┤      └─────────────────────┘
                     │ + request()         │
                     │   adaptee.          │
                     │   specificRequest() │
                     └─────────────────────┘
```

### Class Adapter (Using Multiple Inheritance)

```
┌─────────────────────┐                    ┌─────────────────────┐
│      <<interface>>  │                    │      Adaptee        │
│        Target       │                    ├─────────────────────┤
├─────────────────────┤                    │ + specificRequest() │
│ + request()         │                    └─────────────────────┘
└─────────────────────┘                              △
          △                                          │ extends
          │ implements                               │
          │              ┌─────────────────────┐     │
          └──────────────│      Adapter        │─────┘
                         ├─────────────────────┤
                         │ + request()         │
                         │   specificRequest() │
                         └─────────────────────┘
```

## Participants

- **Target**: Defines the domain-specific interface that the Client uses. This is the interface that the client code expects and is designed to work with.

- **Client**: Collaborates with objects conforming to the Target interface. The client code does not need to know whether it is working with a real target or an adapter.

- **Adaptee**: Defines an existing interface that needs adapting. This is often a third-party class, legacy code, or a class from a different subsystem with an incompatible interface.

- **Adapter**: Adapts the interface of Adaptee to the Target interface. It translates requests from the client into requests that the adaptee understands.

## Collaborations

1. Clients call operations on an Adapter instance.

2. The Adapter translates the request into one or more calls on the Adaptee using its specific interface.

3. The Adapter may perform additional work such as data transformation, parameter conversion, or result formatting to complete the adaptation.

4. The result is returned to the Client in the expected format of the Target interface.

The client remains decoupled from the adaptee and can work with any object that conforms to the target interface, whether it is a native implementation or an adapter wrapping a foreign object.

## Consequences

### Benefits

- **Single Responsibility Principle**: You can separate the interface conversion code from the primary business logic of the program. The adapter handles all the translation, keeping both the client and adaptee focused on their core responsibilities.

- **Open/Closed Principle**: You can introduce new types of adapters into the program without breaking existing client code, as long as they work with adapters through the target interface.

- **Reusability**: Existing classes can be reused even if they have incompatible interfaces. This is especially valuable for third-party libraries and legacy code.

- **Flexibility**: The object adapter lets a single adapter work with multiple adaptees (the adaptee itself and all of its subclasses). The adapter can also add functionality to all adaptees at once.

- **Testability**: Adapters make it easier to test client code in isolation by allowing you to substitute mock adapters for real ones.

- **Gradual Migration**: Adapters enable gradual migration from one interface to another, allowing you to introduce changes incrementally.

### Liabilities

- **Increased Complexity**: The overall complexity of the code increases because you need to introduce a set of new interfaces and classes. Sometimes it is simpler to just change the service class so that it matches the rest of your code.

- **Performance Overhead**: Each call goes through an additional layer of indirection. While usually negligible, this can matter in performance-critical code paths.

- **Class Adapter Limitations**: A class adapter will not work when you need to adapt a class and all its subclasses, since it commits to a concrete adaptee class at compile time.

- **Two-Way Adapters Complexity**: When you need to make the adapter work both ways (allowing clients of both interfaces to use either object), the adapter becomes more complex and harder to maintain.

- **Hidden Dependencies**: Adapters can hide the true dependencies of a system, making it harder to understand what the client code actually depends on.

## Implementation

### Implementation Considerations

1. **Class Adapter vs Object Adapter**: In languages that support multiple inheritance, you can use a class adapter that inherits from both the target and the adaptee. In single-inheritance languages (most common), use an object adapter that holds a reference to the adaptee.

2. **Adapter Granularity**: Decide whether the adapter should adapt a single method, a group of related methods, or an entire class interface. Smaller adapters are more reusable but require more classes.

3. **Pluggable Adapters**: Build adapters with flexibility in mind by parameterizing the adaptation logic. This allows the same adapter class to work with different adaptees.

4. **Two-Way Adapters**: When both interfaces have methods that need to be exposed, consider creating a two-way adapter that implements both interfaces.

5. **Null Adapters**: Consider creating null adapters that provide default behavior when no real adaptee is available.

### Basic Object Adapter Implementation

```
// Target interface that the client expects
interface MediaPlayer
    method play(audioType: String, fileName: String)
end interface

// Adaptee - existing class with incompatible interface
class AdvancedMediaPlayer
    method playVlc(fileName: String)
        // VLC-specific playback logic
        print "Playing VLC file: " + fileName
    end method

    method playMp4(fileName: String)
        // MP4-specific playback logic
        print "Playing MP4 file: " + fileName
    end method
end class

// Adapter - makes AdvancedMediaPlayer compatible with MediaPlayer
class MediaAdapter implements MediaPlayer
    private advancedPlayer: AdvancedMediaPlayer

    constructor(audioType: String)
        advancedPlayer = new AdvancedMediaPlayer()
    end constructor

    method play(audioType: String, fileName: String)
        if audioType equals "vlc"
            advancedPlayer.playVlc(fileName)
        else if audioType equals "mp4"
            advancedPlayer.playMp4(fileName)
        end if
    end method
end class

// Client - uses MediaPlayer interface
class AudioPlayer implements MediaPlayer
    private mediaAdapter: MediaAdapter

    method play(audioType: String, fileName: String)
        // Built-in support for mp3
        if audioType equals "mp3"
            print "Playing MP3 file: " + fileName
        // Adapter provides support for other formats
        else if audioType equals "vlc" or audioType equals "mp4"
            mediaAdapter = new MediaAdapter(audioType)
            mediaAdapter.play(audioType, fileName)
        else
            print "Invalid media type: " + audioType
        end if
    end method
end class
```

### Pluggable Adapter Implementation

```
// Generic adapter that can adapt any function to the target interface
class PluggableAdapter implements Target
    private adaptedFunction: Function
    private parameterMapper: Function
    private resultMapper: Function

    constructor(
        adaptedFunction: Function,
        parameterMapper: Function,
        resultMapper: Function
    )
        this.adaptedFunction = adaptedFunction
        this.parameterMapper = parameterMapper
        this.resultMapper = resultMapper
    end constructor

    method request(params: Object): Object
        // Map parameters to adaptee's expected format
        mappedParams = parameterMapper(params)

        // Call the adapted function
        result = adaptedFunction(mappedParams)

        // Map result back to target's expected format
        return resultMapper(result)
    end method
end class

// Usage
legacyFunction = someOldLibrary.doSomething
adapter = new PluggableAdapter(
    legacyFunction,
    params -> convertToLegacyFormat(params),
    result -> convertFromLegacyFormat(result)
)
```

### Two-Way Adapter Implementation

```
// Interface A
interface DatabaseConnection
    method query(sql: String): ResultSet
    method execute(sql: String): Integer
end interface

// Interface B
interface DocumentStore
    method find(criteria: Object): List
    method save(document: Object): String
end interface

// Two-way adapter that works with both interfaces
class DatabaseDocumentAdapter implements DatabaseConnection, DocumentStore
    private database: RealDatabase
    private converter: QueryConverter

    constructor(database: RealDatabase)
        this.database = database
        this.converter = new QueryConverter()
    end constructor

    // DatabaseConnection interface
    method query(sql: String): ResultSet
        return database.executeQuery(sql)
    end method

    method execute(sql: String): Integer
        return database.executeUpdate(sql)
    end method

    // DocumentStore interface
    method find(criteria: Object): List
        sql = converter.criteriaToSQL(criteria)
        resultSet = database.executeQuery(sql)
        return converter.resultSetToDocuments(resultSet)
    end method

    method save(document: Object): String
        sql = converter.documentToInsertSQL(document)
        database.executeUpdate(sql)
        return document.getId()
    end method
end class
```

## Example

### Payment Gateway Integration

A real-world scenario where an e-commerce application needs to integrate with multiple payment providers, each having their own API.

```
// Target interface - what our application expects
interface PaymentProcessor
    method processPayment(
        amount: Decimal,
        currency: String,
        cardNumber: String,
        expiryDate: String,
        cvv: String
    ): PaymentResult

    method refund(transactionId: String, amount: Decimal): RefundResult

    method getTransactionStatus(transactionId: String): TransactionStatus
end interface

class PaymentResult
    public success: Boolean
    public transactionId: String
    public errorMessage: String
    public providerResponse: Object
end class

// Adaptee 1: Stripe-like API
class StripeAPI
    method createCharge(params: Object): Object
        // params = { amount_cents, currency, source: { number, exp, cvc } }
        // Returns { id, status, failure_message }
        print "Stripe: Creating charge for " + params.amount_cents + " cents"
        return {
            id: "ch_stripe_" + generateId(),
            status: "succeeded",
            failure_message: null
        }
    end method

    method createRefund(chargeId: String, amountCents: Integer): Object
        print "Stripe: Refunding " + amountCents + " cents for charge " + chargeId
        return { id: "re_" + generateId(), status: "succeeded" }
    end method

    method retrieveCharge(chargeId: String): Object
        return { id: chargeId, status: "succeeded" }
    end method
end class

// Adaptee 2: PayPal-like API
class PayPalSDK
    method createOrder(orderData: Object): Object
        // Completely different structure
        // orderData = { intent, purchase_units: [{ amount: { value, currency_code } }] }
        print "PayPal: Creating order for $" + orderData.purchase_units[0].amount.value
        return { id: "PAY-" + generateId(), status: "CREATED" }
    end method

    method captureOrder(orderId: String, cardDetails: Object): Object
        print "PayPal: Capturing order " + orderId
        return {
            id: orderId,
            status: "COMPLETED",
            purchase_units: [{ payments: { captures: [{ id: "CAP-" + generateId() }] } }]
        }
    end method

    method refundCapture(captureId: String, refundData: Object): Object
        print "PayPal: Refunding capture " + captureId
        return { id: "REF-" + generateId(), status: "COMPLETED" }
    end method

    method getOrderDetails(orderId: String): Object
        return { id: orderId, status: "COMPLETED" }
    end method
end class

// Adapter for Stripe
class StripeAdapter implements PaymentProcessor
    private stripe: StripeAPI

    constructor(apiKey: String)
        this.stripe = new StripeAPI()
        // Configure with API key
    end constructor

    method processPayment(
        amount: Decimal,
        currency: String,
        cardNumber: String,
        expiryDate: String,
        cvv: String
    ): PaymentResult
        // Convert to Stripe's expected format
        stripeParams = {
            amount_cents: convertToCents(amount),
            currency: currency.toLowerCase(),
            source: {
                number: cardNumber,
                exp: expiryDate,
                cvc: cvv
            }
        }

        try
            response = stripe.createCharge(stripeParams)

            result = new PaymentResult()
            result.success = (response.status == "succeeded")
            result.transactionId = response.id
            result.errorMessage = response.failure_message
            result.providerResponse = response

            return result
        catch error
            result = new PaymentResult()
            result.success = false
            result.errorMessage = error.message
            return result
        end try
    end method

    method refund(transactionId: String, amount: Decimal): RefundResult
        amountCents = convertToCents(amount)
        response = stripe.createRefund(transactionId, amountCents)

        result = new RefundResult()
        result.success = (response.status == "succeeded")
        result.refundId = response.id
        return result
    end method

    method getTransactionStatus(transactionId: String): TransactionStatus
        response = stripe.retrieveCharge(transactionId)
        return mapStripeStatus(response.status)
    end method

    private method convertToCents(amount: Decimal): Integer
        return Math.round(amount * 100)
    end method

    private method mapStripeStatus(stripeStatus: String): TransactionStatus
        statusMap = {
            "succeeded": TransactionStatus.COMPLETED,
            "pending": TransactionStatus.PENDING,
            "failed": TransactionStatus.FAILED
        }
        return statusMap[stripeStatus] or TransactionStatus.UNKNOWN
    end method
end class

// Adapter for PayPal
class PayPalAdapter implements PaymentProcessor
    private paypal: PayPalSDK
    private orderToCapture: Map  // Store mapping for refunds

    constructor(clientId: String, clientSecret: String)
        this.paypal = new PayPalSDK()
        this.orderToCapture = new Map()
        // Configure with credentials
    end constructor

    method processPayment(
        amount: Decimal,
        currency: String,
        cardNumber: String,
        expiryDate: String,
        cvv: String
    ): PaymentResult
        // PayPal requires a two-step process: create order, then capture

        // Step 1: Create order
        orderData = {
            intent: "CAPTURE",
            purchase_units: [{
                amount: {
                    value: amount.toString(),
                    currency_code: currency.toUpperCase()
                }
            }]
        }

        try
            orderResponse = paypal.createOrder(orderData)

            // Step 2: Capture the order
            cardDetails = {
                number: cardNumber,
                expiry: expiryDate,
                security_code: cvv
            }

            captureResponse = paypal.captureOrder(orderResponse.id, cardDetails)

            // Extract capture ID for potential refunds
            captureId = captureResponse.purchase_units[0].payments.captures[0].id
            orderToCapture.set(orderResponse.id, captureId)

            result = new PaymentResult()
            result.success = (captureResponse.status == "COMPLETED")
            result.transactionId = orderResponse.id
            result.providerResponse = captureResponse

            return result
        catch error
            result = new PaymentResult()
            result.success = false
            result.errorMessage = error.message
            return result
        end try
    end method

    method refund(transactionId: String, amount: Decimal): RefundResult
        // Need to refund the capture, not the order
        captureId = orderToCapture.get(transactionId)

        refundData = {
            amount: {
                value: amount.toString(),
                currency_code: "USD"  // Would need to store this
            }
        }

        response = paypal.refundCapture(captureId, refundData)

        result = new RefundResult()
        result.success = (response.status == "COMPLETED")
        result.refundId = response.id
        return result
    end method

    method getTransactionStatus(transactionId: String): TransactionStatus
        response = paypal.getOrderDetails(transactionId)
        return mapPayPalStatus(response.status)
    end method

    private method mapPayPalStatus(paypalStatus: String): TransactionStatus
        statusMap = {
            "COMPLETED": TransactionStatus.COMPLETED,
            "CREATED": TransactionStatus.PENDING,
            "APPROVED": TransactionStatus.PENDING,
            "VOIDED": TransactionStatus.FAILED
        }
        return statusMap[paypalStatus] or TransactionStatus.UNKNOWN
    end method
end class

// Client code - works with any payment processor
class CheckoutService
    private paymentProcessor: PaymentProcessor

    constructor(paymentProcessor: PaymentProcessor)
        this.paymentProcessor = paymentProcessor
    end constructor

    method checkout(cart: ShoppingCart, paymentDetails: PaymentDetails): Order
        totalAmount = cart.calculateTotal()

        result = paymentProcessor.processPayment(
            totalAmount,
            "USD",
            paymentDetails.cardNumber,
            paymentDetails.expiryDate,
            paymentDetails.cvv
        )

        if result.success
            order = createOrder(cart, result.transactionId)
            return order
        else
            throw new PaymentFailedException(result.errorMessage)
        end if
    end method

    method processRefund(order: Order, amount: Decimal): Boolean
        result = paymentProcessor.refund(order.transactionId, amount)

        if result.success
            order.addRefund(result.refundId, amount)
            return true
        end if

        return false
    end method
end class

// Usage - easily switch between payment providers
stripeProcessor = new StripeAdapter("sk_test_...")
paypalProcessor = new PayPalAdapter("client_id", "client_secret")

// Use Stripe
checkoutService = new CheckoutService(stripeProcessor)
order = checkoutService.checkout(cart, paymentDetails)

// Or use PayPal - client code unchanged
checkoutService = new CheckoutService(paypalProcessor)
order = checkoutService.checkout(cart, paymentDetails)

// Factory for selecting processor based on configuration
class PaymentProcessorFactory
    method createProcessor(provider: String): PaymentProcessor
        switch provider
            case "stripe":
                return new StripeAdapter(config.stripeApiKey)
            case "paypal":
                return new PayPalAdapter(config.paypalClientId, config.paypalSecret)
            case "square":
                return new SquareAdapter(config.squareAccessToken)
            default:
                throw new UnknownPaymentProviderException(provider)
        end switch
    end method
end class
```

## Known Uses

- **Java I/O Streams**: `InputStreamReader` adapts an `InputStream` (bytes) to a `Reader` (characters), and `OutputStreamWriter` adapts an `OutputStream` to a `Writer`.

- **Java Collections**: `Arrays.asList()` adapts an array to the `List` interface, allowing arrays to be used wherever lists are expected.

- **Spring Framework**: `HandlerAdapter` adapts different types of handler objects to work with the `DispatcherServlet`, allowing controllers, servlets, and other handlers to be treated uniformly.

- **SLF4J Logging**: Provides adapters for various logging frameworks (Log4j, java.util.logging, Logback) so that application code can use a single logging API.

- **JDBC Drivers**: Database vendors provide JDBC drivers that adapt their proprietary database protocols to the standard JDBC interface.

- **React Native Bridge**: Adapts native mobile APIs (iOS/Android) to JavaScript interfaces that React Native code can call.

- **Retrofit (Android)**: Adapts HTTP APIs to Java/Kotlin interfaces, converting HTTP responses into domain objects.

- **DOM Wrappers**: jQuery and similar libraries adapt the native DOM API to a simpler, more consistent interface that works across browsers.

- **GraphQL Resolvers**: Adapt various data sources (REST APIs, databases, services) to GraphQL's query interface.

- **Payment Gateways**: Libraries like Omnipay (PHP) and ActiveMerchant (Ruby) provide unified interfaces adapting multiple payment provider APIs.

## Related Patterns

- **Bridge**: Has a similar structure to Adapter but a different intent. Bridge separates an abstraction from its implementation so both can vary independently. Adapter makes unrelated classes work together.

- **Decorator**: Enhances an object without changing its interface. Adapter changes the interface of an existing object. Decorator is more transparent to the client.

- **Proxy**: Provides a surrogate for another object but does not change its interface. Adapter provides a different interface to the object it adapts.

- **Facade**: Defines a new interface for an entire subsystem of objects. Adapter reuses an existing interface and makes one interface work with another.

- **Strategy**: Both patterns use composition and delegation. Strategy defines a family of interchangeable algorithms; Adapter makes incompatible interfaces compatible.

- **Repository**: Often uses Adapter internally to abstract away different data sources (SQL, NoSQL, APIs) behind a uniform interface.

## When NOT to Use

- **When interfaces are already compatible**: If the existing class already matches your needs, adding an adapter is unnecessary complexity. Do not wrap classes just because you might need flexibility later.

- **When modification is feasible**: If you can modify the adaptee's source code and no other code depends on its current interface, it may be simpler to change the class directly rather than creating an adapter.

- **For simple one-off uses**: If you only need to call an incompatible method once, a simple inline conversion may be clearer than creating a full adapter class.

- **When performance is critical**: In tight loops or high-frequency code paths, the extra layer of indirection may have measurable overhead. Measure before deciding.

- **When hiding too much complexity**: If the adapter hides important aspects of the adaptee that clients need to understand (error handling, resource management), it may lead to misuse.

- **For speculative flexibility**: Do not create adapters for hypothetical future requirements. Wait until you actually have multiple implementations that need a common interface.

- **When the adaptee is unstable**: If the adaptee's interface changes frequently, you will constantly need to update the adapter. Consider whether the instability can be addressed at the source.

- **Instead of proper design**: An adapter should not be used to paper over fundamental design problems. If you find yourself creating many adapters, consider whether the underlying architecture needs refactoring.

---

## Summary

The Adapter pattern is one of the most practical and widely used structural patterns. It enables integration of disparate systems, reuse of existing code, and flexibility in working with third-party libraries. The key is to use it when you genuinely need to make incompatible interfaces work together, not as a general abstraction layer. When used appropriately, adapters make codebases more modular and easier to maintain; when overused, they add unnecessary complexity.

---

*Based on concepts from "Design Patterns: Elements of Reusable Object-Oriented Software" by Gamma, Helm, Johnson, and Vlissides (Gang of Four), 1994.*
