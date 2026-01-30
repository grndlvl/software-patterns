# Facade

## Intent

Provide a unified interface to a set of interfaces in a subsystem. Facade defines a higher-level interface that makes the subsystem easier to use. It shields clients from the complexity of subsystem components by providing a single, simplified entry point.

## Also Known As

- Wrapper (in the context of wrapping a subsystem)

## Motivation

Structuring a system into subsystems helps reduce complexity. A common design goal is to minimize communication and dependencies between subsystems. One way to achieve this goal is to introduce a facade object that provides a single, simplified interface to the more general facilities of a subsystem.

Consider a compiler subsystem that includes classes like `Scanner`, `Parser`, `ProgramNode`, `BytecodeStream`, and `ProgramNodeBuilder`. Some specialized applications might need to access these classes directly. But most clients of a compiler generally do not care about details like parsing and code generation; they just want to compile some code. For them, the powerful but low-level interfaces in the compiler subsystem only complicate their task.

To provide a higher-level interface that shields clients from these classes, the compiler subsystem includes a `Compiler` facade class. This class defines a unified interface to the compiler's functionality. The `Compiler` class acts as a facade: It offers clients a single, simple interface to the compiler subsystem. It glues together the classes that implement compiler functionality without hiding them completely.

The facade does not add new functionality; it merely simplifies the interface. Subsystem classes remain accessible for clients that need lower-level control. This is a key distinction: the facade is an optional convenience, not a mandatory gateway.

## Applicability

Use the Facade pattern when:

- You want to provide a simple interface to a complex subsystem. Subsystems often get more complex as they evolve. Most patterns, when applied, result in more and smaller classes. This makes the subsystem more reusable and easier to customize, but it also becomes harder to use for clients that do not need to customize it.

- There are many dependencies between clients and the implementation classes of an abstraction. Introduce a facade to decouple the subsystem from clients and other subsystems, promoting independence and portability.

- You want to layer your subsystems. Use a facade to define an entry point to each subsystem level. If subsystems are dependent, you can simplify the dependencies between them by making them communicate solely through their facades.

- You need to wrap a poorly designed API (legacy code, third-party library) with a better-designed interface.

- You want to reduce the learning curve for new developers by providing a clear starting point for common operations.

## Structure

```
┌───────────────────────────────────────────────────────────────────┐
│                           Client                                   │
└───────────────────────────────────────────────────────────────────┘
                                │
                                │ uses
                                ▼
┌───────────────────────────────────────────────────────────────────┐
│                           Facade                                   │
├───────────────────────────────────────────────────────────────────┤
│  + simpleOperation()                                               │
│  + anotherOperation()                                              │
└───────────────────────────────────────────────────────────────────┘
        │              │              │              │
        │ delegates    │              │              │
        ▼              ▼              ▼              ▼
┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│  Subsystem  │ │  Subsystem  │ │  Subsystem  │ │  Subsystem  │
│   Class A   │ │   Class B   │ │   Class C   │ │   Class D   │
└─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘
        │                              │
        └──────────────┬───────────────┘
                       │
                       ▼
              ┌─────────────────┐
              │  Subsystem      │
              │  Class E        │
              └─────────────────┘
```

### Detailed Example Structure

```
┌─────────────────────────────────────────────────────────────────────────┐
│                             Client Code                                  │
│                                                                          │
│   order = orderFacade.placeOrder(customerId, items, paymentInfo)        │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                           OrderFacade                                    │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │  placeOrder(customerId, items, paymentInfo):                      │  │
│  │    1. inventory.checkAvailability(items)                          │  │
│  │    2. pricing.calculateTotal(items)                               │  │
│  │    3. payment.processPayment(total, paymentInfo)                  │  │
│  │    4. inventory.reserveItems(items)                               │  │
│  │    5. shipping.createShipment(items, address)                     │  │
│  │    6. notification.sendConfirmation(customer, order)              │  │
│  │    7. return order                                                │  │
│  └───────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
           │           │           │           │           │
           ▼           ▼           ▼           ▼           ▼
    ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
    │Inventory │ │ Pricing  │ │ Payment  │ │ Shipping │ │Notification│
    │ Service  │ │ Engine   │ │ Gateway  │ │ Service  │ │  Service  │
    └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘
```

## Participants

- **Facade**: Knows which subsystem classes are responsible for a request. Delegates client requests to appropriate subsystem objects. May perform additional coordination work.

- **Subsystem Classes**: Implement subsystem functionality. Handle work assigned by the Facade object. Have no knowledge of the facade; they keep no reference to it. They can be used directly by clients who need finer-grained control.

- **Client**: Interacts with the subsystem through the Facade. Can also access subsystem classes directly when needed.

## Collaborations

1. Clients communicate with the subsystem by sending requests to the Facade.

2. The Facade translates high-level requests into one or more calls to subsystem objects.

3. The Facade may need to do additional work to coordinate subsystem interactions.

4. Clients that use the Facade do not have to access subsystem objects directly.

5. Clients can still access subsystem objects directly when they need lower-level functionality.

## Consequences

### Benefits

- **Shields clients from subsystem complexity**: Clients deal with a simple interface rather than many subsystem classes. Reduces the number of objects clients deal with.

- **Promotes weak coupling**: Decouples the subsystem from its clients and other subsystems. Subsystem components can change without affecting clients that use the facade.

- **Does not prevent direct access**: Clients can still use subsystem classes directly when they need to. The facade is an optional convenience, not a mandatory gateway.

- **Layers the subsystem**: Facades help define entry points to each layer of a subsystem. When subsystems depend on each other, facades simplify inter-subsystem communication.

- **Simplifies porting and testing**: When you port to a new platform, only the facade needs to change if the subsystem interface differs.

- **Provides context for operations**: The facade can add context, logging, error handling, and transaction management around subsystem operations.

### Liabilities

- **Can become a god object**: If the facade takes on too much responsibility, it can become a monolithic class that knows too much about the subsystem.

- **Additional layer**: Adds another layer of abstraction. For simple subsystems, this may be unnecessary overhead.

- **May hide important details**: If the facade oversimplifies, clients may miss important nuances of subsystem behavior.

- **Maintenance burden**: The facade must be updated when subsystem interfaces change. It becomes another artifact to maintain.

- **Can mask poor design**: A facade can paper over a poorly designed subsystem without fixing the underlying problems.

## Implementation

### Implementation Considerations

1. **Reducing client-subsystem coupling**: You can further reduce coupling by making the Facade an abstract class with concrete subclasses for different implementations of the subsystem. Clients can communicate with the subsystem through the interface of the abstract Facade class.

2. **Public vs private subsystem classes**: Consider which subsystem classes should be public. Making classes public exposes them to clients who might bypass the facade. In languages with package/module visibility, you can hide subsystem classes.

3. **Single facade vs multiple facades**: Large subsystems may benefit from multiple facades, each providing a cohesive set of operations for different aspects of the subsystem.

4. **Facade as coordinator**: The facade can do more than just delegate. It can coordinate calls, handle transactions, and manage error recovery.

### Basic Facade Implementation

```
// Subsystem classes - complex but powerful
class VideoFile
    private filename: String
    private codecType: String
    private data: Bytes

    constructor(filename: String)
        this.filename = filename
        this.codecType = detectCodec(filename)
        this.data = readFile(filename)
    end constructor

    method getCodecType(): String
        return codecType
    end method

    method getData(): Bytes
        return data
    end method
end class

class CodecFactory
    method extract(file: VideoFile): Codec
        codecType = file.getCodecType()
        switch codecType
            case "mp4":
                return new MPEG4CompressionCodec()
            case "ogg":
                return new OggCompressionCodec()
            default:
                throw new UnsupportedCodecException(codecType)
        end switch
    end method
end class

class MPEG4CompressionCodec implements Codec
    method decode(data: Bytes): VideoBuffer
        // Complex MPEG4 decoding logic
    end method

    method encode(buffer: VideoBuffer): Bytes
        // Complex MPEG4 encoding logic
    end method
end class

class OggCompressionCodec implements Codec
    method decode(data: Bytes): VideoBuffer
        // Complex Ogg decoding logic
    end method

    method encode(buffer: VideoBuffer): Bytes
        // Complex Ogg encoding logic
    end method
end class

class BitrateReader
    method read(file: VideoFile, codec: Codec): VideoBuffer
        // Read video data and decode
        return codec.decode(file.getData())
    end method

    method convert(buffer: VideoBuffer, codec: Codec): Bytes
        // Encode video data
        return codec.encode(buffer)
    end method
end class

class AudioMixer
    method fix(buffer: VideoBuffer): VideoBuffer
        // Normalize audio levels
        // Remove noise
        // Sync audio with video
        return processedBuffer
    end method
end class

// Facade - simple interface for video conversion
class VideoConverter
    method convert(filename: String, targetFormat: String): File
        // Load the source file
        file = new VideoFile(filename)

        // Get appropriate codecs
        codecFactory = new CodecFactory()
        sourceCodec = codecFactory.extract(file)

        // Create target codec
        targetCodec = createCodecForFormat(targetFormat)

        // Read and decode source
        reader = new BitrateReader()
        buffer = reader.read(file, sourceCodec)

        // Fix audio
        audioMixer = new AudioMixer()
        buffer = audioMixer.fix(buffer)

        // Encode to target format
        result = reader.convert(buffer, targetCodec)

        // Write output file
        outputFilename = replaceExtension(filename, targetFormat)
        return writeFile(outputFilename, result)
    end method

    private method createCodecForFormat(format: String): Codec
        switch format
            case "mp4":
                return new MPEG4CompressionCodec()
            case "ogg":
                return new OggCompressionCodec()
            default:
                throw new UnsupportedFormatException(format)
        end switch
    end method
end class

// Client code - simple and clean
converter = new VideoConverter()
mp4File = converter.convert("birthday.ogg", "mp4")
print "Converted video saved to: " + mp4File.getName()
```

### Facade with Multiple Entry Points

```
// Large subsystem with multiple facades for different concerns
// Subsystem: E-commerce platform

// === Subsystem Classes ===
class ProductCatalog
    method search(query: String, filters: Filters): List<Product>
    method getProduct(id: String): Product
    method getRelatedProducts(product: Product): List<Product>
    method getCategories(): List<Category>
end class

class InventoryManager
    method checkStock(productId: String): Integer
    method reserveStock(productId: String, quantity: Integer): Reservation
    method releaseReservation(reservation: Reservation)
    method updateStock(productId: String, quantity: Integer)
end class

class PricingEngine
    method getPrice(product: Product): Money
    method calculateDiscount(products: List<Product>, coupons: List<Coupon>): Discount
    method applyTaxes(subtotal: Money, address: Address): Money
end class

class ShoppingCart
    method addItem(userId: String, product: Product, quantity: Integer)
    method removeItem(userId: String, productId: String)
    method getItems(userId: String): List<CartItem>
    method clear(userId: String)
end class

class PaymentProcessor
    method authorize(amount: Money, paymentMethod: PaymentMethod): Authorization
    method capture(authorization: Authorization): Payment
    method refund(payment: Payment, amount: Money): Refund
    method void(authorization: Authorization)
end class

class OrderManager
    method createOrder(customer: Customer, items: List<OrderItem>, payment: Payment): Order
    method updateStatus(orderId: String, status: OrderStatus)
    method cancelOrder(orderId: String): Cancellation
    method getOrder(orderId: String): Order
end class

class ShippingService
    method calculateShippingOptions(items: List<Item>, address: Address): List<ShippingOption>
    method createShipment(order: Order, option: ShippingOption): Shipment
    method trackShipment(trackingNumber: String): TrackingInfo
end class

class NotificationService
    method sendEmail(recipient: String, template: String, data: Map)
    method sendSMS(phone: String, message: String)
    method sendPush(userId: String, notification: Notification)
end class

class CustomerService
    method getCustomer(id: String): Customer
    method updateCustomer(customer: Customer)
    method getAddresses(customerId: String): List<Address>
    method getPaymentMethods(customerId: String): List<PaymentMethod>
end class

// === Facade 1: Shopping Facade ===
// For browsing and cart operations
class ShoppingFacade
    private catalog: ProductCatalog
    private inventory: InventoryManager
    private pricing: PricingEngine
    private cart: ShoppingCart

    constructor(
        catalog: ProductCatalog,
        inventory: InventoryManager,
        pricing: PricingEngine,
        cart: ShoppingCart
    )
        this.catalog = catalog
        this.inventory = inventory
        this.pricing = pricing
        this.cart = cart
    end constructor

    // Simple product search with availability
    method searchProducts(query: String): List<ProductResult>
        products = catalog.search(query, Filters.default())

        results = new List()
        for each product in products
            price = pricing.getPrice(product)
            stock = inventory.checkStock(product.id)

            results.add(new ProductResult(
                product: product,
                price: price,
                inStock: stock > 0,
                stockLevel: stock
            ))
        end for

        return results
    end method

    // Get product details with recommendations
    method getProductDetails(productId: String): ProductDetails
        product = catalog.getProduct(productId)
        price = pricing.getPrice(product)
        stock = inventory.checkStock(productId)
        related = catalog.getRelatedProducts(product)

        return new ProductDetails(
            product: product,
            price: price,
            inStock: stock > 0,
            relatedProducts: related.take(4)
        )
    end method

    // Add to cart with validation
    method addToCart(userId: String, productId: String, quantity: Integer): CartResult
        // Check if product exists and is in stock
        product = catalog.getProduct(productId)
        stock = inventory.checkStock(productId)

        if stock < quantity
            return CartResult.failure("Insufficient stock. Only " + stock + " available.")
        end if

        cart.addItem(userId, product, quantity)

        return CartResult.success(getCartSummary(userId))
    end method

    // Get cart with current prices
    method getCartSummary(userId: String): CartSummary
        items = cart.getItems(userId)

        cartItems = new List()
        subtotal = Money.zero()

        for each item in items
            price = pricing.getPrice(item.product)
            lineTotal = price.multiply(item.quantity)

            cartItems.add(new CartItemSummary(
                product: item.product,
                quantity: item.quantity,
                unitPrice: price,
                lineTotal: lineTotal
            ))

            subtotal = subtotal.add(lineTotal)
        end for

        return new CartSummary(
            items: cartItems,
            subtotal: subtotal,
            itemCount: items.size()
        )
    end method
end class

// === Facade 2: Checkout Facade ===
// For the checkout process
class CheckoutFacade
    private cart: ShoppingCart
    private inventory: InventoryManager
    private pricing: PricingEngine
    private payment: PaymentProcessor
    private orders: OrderManager
    private shipping: ShippingService
    private notifications: NotificationService
    private customers: CustomerService

    constructor(/* all dependencies */)
        // Initialize all subsystem references
    end constructor

    // Start checkout - get options and calculate totals
    method initiateCheckout(userId: String): CheckoutSession
        customer = customers.getCustomer(userId)
        cartItems = cart.getItems(userId)

        if cartItems.isEmpty()
            throw new EmptyCartException()
        end if

        // Calculate subtotal
        subtotal = calculateSubtotal(cartItems)

        // Get saved addresses and payment methods
        addresses = customers.getAddresses(userId)
        paymentMethods = customers.getPaymentMethods(userId)

        // Get shipping options for default address
        defaultAddress = addresses.find(a -> a.isDefault)
        shippingOptions = shipping.calculateShippingOptions(cartItems, defaultAddress)

        return new CheckoutSession(
            sessionId: generateSessionId(),
            items: cartItems,
            subtotal: subtotal,
            addresses: addresses,
            paymentMethods: paymentMethods,
            shippingOptions: shippingOptions
        )
    end method

    // Calculate final totals with shipping and tax
    method calculateTotals(
        userId: String,
        shippingAddress: Address,
        shippingOption: ShippingOption,
        coupons: List<String>
    ): OrderTotals
        cartItems = cart.getItems(userId)
        subtotal = calculateSubtotal(cartItems)

        // Apply discounts
        couponObjects = coupons.map(code -> loadCoupon(code))
        discount = pricing.calculateDiscount(
            cartItems.map(i -> i.product),
            couponObjects
        )

        // Calculate shipping
        shippingCost = shippingOption.cost

        // Calculate taxes
        taxableAmount = subtotal.subtract(discount.amount).add(shippingCost)
        taxes = pricing.applyTaxes(taxableAmount, shippingAddress)

        total = taxableAmount.add(taxes)

        return new OrderTotals(
            subtotal: subtotal,
            discount: discount,
            shipping: shippingCost,
            taxes: taxes,
            total: total
        )
    end method

    // Complete the order
    method placeOrder(
        userId: String,
        shippingAddressId: String,
        paymentMethodId: String,
        shippingOptionId: String
    ): OrderConfirmation
        customer = customers.getCustomer(userId)
        cartItems = cart.getItems(userId)
        shippingAddress = customers.getAddress(shippingAddressId)
        paymentMethod = customers.getPaymentMethod(paymentMethodId)
        shippingOption = getShippingOption(shippingOptionId)

        // Calculate final totals
        totals = calculateTotals(userId, shippingAddress, shippingOption, [])

        // Reserve inventory
        reservations = new List()
        try
            for each item in cartItems
                reservation = inventory.reserveStock(item.product.id, item.quantity)
                reservations.add(reservation)
            end for
        catch error
            // Release any successful reservations
            for each reservation in reservations
                inventory.releaseReservation(reservation)
            end for
            throw new OutOfStockException(error.message)
        end try

        // Process payment
        try
            authorization = payment.authorize(totals.total, paymentMethod)
            capturedPayment = payment.capture(authorization)
        catch error
            // Release inventory on payment failure
            for each reservation in reservations
                inventory.releaseReservation(reservation)
            end for
            throw new PaymentFailedException(error.message)
        end try

        // Create order
        order = orders.createOrder(customer, cartItems, capturedPayment)

        // Create shipment
        shipment = shipping.createShipment(order, shippingOption)

        // Clear cart
        cart.clear(userId)

        // Send confirmation
        notifications.sendEmail(
            customer.email,
            "order-confirmation",
            { order: order, shipment: shipment }
        )

        return new OrderConfirmation(
            orderId: order.id,
            orderNumber: order.number,
            total: totals.total,
            estimatedDelivery: shipment.estimatedDelivery,
            trackingNumber: shipment.trackingNumber
        )
    end method

    private method calculateSubtotal(items: List<CartItem>): Money
        total = Money.zero()
        for each item in items
            price = pricing.getPrice(item.product)
            total = total.add(price.multiply(item.quantity))
        end for
        return total
    end method
end class

// === Facade 3: Order Management Facade ===
// For post-purchase operations
class OrderManagementFacade
    private orders: OrderManager
    private payment: PaymentProcessor
    private shipping: ShippingService
    private inventory: InventoryManager
    private notifications: NotificationService
    private customers: CustomerService

    constructor(/* dependencies */)
        // Initialize
    end constructor

    // Get order history with tracking
    method getOrderHistory(userId: String): List<OrderSummary>
        customerOrders = orders.getOrdersForCustomer(userId)

        summaries = new List()
        for each order in customerOrders
            tracking = null
            if order.shipment != null
                tracking = shipping.trackShipment(order.shipment.trackingNumber)
            end if

            summaries.add(new OrderSummary(
                orderId: order.id,
                orderNumber: order.number,
                date: order.createdAt,
                total: order.total,
                status: order.status,
                trackingStatus: tracking?.status
            ))
        end for

        return summaries
    end method

    // Track shipment
    method trackOrder(orderId: String): TrackingDetails
        order = orders.getOrder(orderId)
        tracking = shipping.trackShipment(order.shipment.trackingNumber)

        return new TrackingDetails(
            orderNumber: order.number,
            carrier: order.shipment.carrier,
            trackingNumber: order.shipment.trackingNumber,
            status: tracking.status,
            estimatedDelivery: tracking.estimatedDelivery,
            events: tracking.events
        )
    end method

    // Cancel order
    method cancelOrder(orderId: String, reason: String): CancellationResult
        order = orders.getOrder(orderId)

        // Check if cancellable
        if not order.isCancellable()
            return CancellationResult.failure("Order cannot be cancelled in current status")
        end if

        // Process refund
        refund = payment.refund(order.payment, order.total)

        // Release inventory
        for each item in order.items
            inventory.updateStock(item.productId, item.quantity)
        end for

        // Update order status
        orders.cancelOrder(orderId)

        // Notify customer
        customer = customers.getCustomer(order.customerId)
        notifications.sendEmail(
            customer.email,
            "order-cancelled",
            { order: order, refund: refund, reason: reason }
        )

        return CancellationResult.success(refund.id)
    end method

    // Request return
    method initiateReturn(orderId: String, items: List<ReturnItem>): ReturnRequest
        order = orders.getOrder(orderId)

        // Validate items can be returned
        for each item in items
            if not isReturnable(order, item)
                throw new ItemNotReturnableException(item.productId)
            end if
        end for

        // Create return request
        returnRequest = createReturnRequest(order, items)

        // Generate return label
        returnLabel = shipping.createReturnLabel(order.shipment)

        // Notify customer
        customer = customers.getCustomer(order.customerId)
        notifications.sendEmail(
            customer.email,
            "return-initiated",
            { order: order, returnRequest: returnRequest, returnLabel: returnLabel }
        )

        return returnRequest
    end method
end class

// Client usage - each facade provides a focused API
shoppingFacade = new ShoppingFacade(catalog, inventory, pricing, cart)
checkoutFacade = new CheckoutFacade(/* ... */)
orderFacade = new OrderManagementFacade(/* ... */)

// Shopping
results = shoppingFacade.searchProducts("laptop")
shoppingFacade.addToCart(userId, "prod-123", 1)
cartSummary = shoppingFacade.getCartSummary(userId)

// Checkout
session = checkoutFacade.initiateCheckout(userId)
totals = checkoutFacade.calculateTotals(userId, address, shippingOption, coupons)
confirmation = checkoutFacade.placeOrder(userId, addressId, paymentId, shippingId)

// Order management
history = orderFacade.getOrderHistory(userId)
tracking = orderFacade.trackOrder(orderId)
```

## Example

### Home Theater System

A classic example where multiple complex components need to be coordinated for a simple user experience.

```
// Subsystem components
class Amplifier
    private tuner: Tuner
    private player: StreamingPlayer
    private volume: Integer

    method on()
        print "Amplifier on"
    end method

    method off()
        print "Amplifier off"
    end method

    method setStreamingPlayer(player: StreamingPlayer)
        this.player = player
        print "Amplifier setting streaming player"
    end method

    method setTuner(tuner: Tuner)
        this.tuner = tuner
    end method

    method setStereoSound()
        print "Amplifier stereo mode on"
    end method

    method setSurroundSound()
        print "Amplifier surround sound on (5.1)"
    end method

    method setVolume(level: Integer)
        this.volume = level
        print "Amplifier volume set to " + level
    end method
end class

class Tuner
    private amplifier: Amplifier
    private frequency: Float

    constructor(amplifier: Amplifier)
        this.amplifier = amplifier
    end constructor

    method on()
        print "Tuner on"
    end method

    method off()
        print "Tuner off"
    end method

    method setFrequency(frequency: Float)
        this.frequency = frequency
        print "Tuner frequency set to " + frequency
    end method

    method setAm()
        print "Tuner AM mode"
    end method

    method setFm()
        print "Tuner FM mode"
    end method
end class

class StreamingPlayer
    private amplifier: Amplifier
    private currentMovie: String

    constructor(amplifier: Amplifier)
        this.amplifier = amplifier
    end constructor

    method on()
        print "Streaming Player on"
    end method

    method off()
        print "Streaming Player off"
    end method

    method play(movie: String)
        this.currentMovie = movie
        print "Streaming Player playing \"" + movie + "\""
    end method

    method pause()
        print "Streaming Player paused \"" + currentMovie + "\""
    end method

    method stop()
        print "Streaming Player stopped \"" + currentMovie + "\""
    end method

    method setSurroundAudio()
        print "Streaming Player surround audio on"
    end method

    method setTwoChannelAudio()
        print "Streaming Player 2-channel audio on"
    end method
end class

class Projector
    private player: StreamingPlayer

    constructor(player: StreamingPlayer)
        this.player = player
    end constructor

    method on()
        print "Projector on"
    end method

    method off()
        print "Projector off"
    end method

    method wideScreenMode()
        print "Projector in widescreen mode (16:9)"
    end method

    method tvMode()
        print "Projector in TV mode (4:3)"
    end method
end class

class Screen
    method down()
        print "Screen going down"
    end method

    method up()
        print "Screen going up"
    end method
end class

class TheaterLights
    private level: Integer

    method on()
        this.level = 100
        print "Theater lights on"
    end method

    method off()
        this.level = 0
        print "Theater lights off"
    end method

    method dim(level: Integer)
        this.level = level
        print "Theater lights dimming to " + level + "%"
    end method
end class

class PopcornPopper
    method on()
        print "Popcorn Popper on"
    end method

    method off()
        print "Popcorn Popper off"
    end method

    method pop()
        print "Popcorn Popper popping popcorn!"
    end method
end class

// Without Facade - client must coordinate everything
// This is complex and error-prone
method watchMovieWithoutFacade(movie: String)
    print "Get ready to watch a movie..."
    popper.on()
    popper.pop()
    lights.dim(10)
    screen.down()
    projector.on()
    projector.wideScreenMode()
    amp.on()
    amp.setStreamingPlayer(player)
    amp.setSurroundSound()
    amp.setVolume(5)
    player.on()
    player.setSurroundAudio()
    player.play(movie)
end method

// Facade - simplifies the complex coordination
class HomeTheaterFacade
    private amp: Amplifier
    private tuner: Tuner
    private player: StreamingPlayer
    private projector: Projector
    private screen: Screen
    private lights: TheaterLights
    private popper: PopcornPopper

    constructor(
        amp: Amplifier,
        tuner: Tuner,
        player: StreamingPlayer,
        projector: Projector,
        screen: Screen,
        lights: TheaterLights,
        popper: PopcornPopper
    )
        this.amp = amp
        this.tuner = tuner
        this.player = player
        this.projector = projector
        this.screen = screen
        this.lights = lights
        this.popper = popper
    end constructor

    method watchMovie(movie: String)
        print "Get ready to watch a movie..."
        popper.on()
        popper.pop()
        lights.dim(10)
        screen.down()
        projector.on()
        projector.wideScreenMode()
        amp.on()
        amp.setStreamingPlayer(player)
        amp.setSurroundSound()
        amp.setVolume(5)
        player.on()
        player.setSurroundAudio()
        player.play(movie)
    end method

    method endMovie()
        print "Shutting down the theater..."
        popper.off()
        lights.on()
        screen.up()
        projector.off()
        amp.off()
        player.stop()
        player.off()
    end method

    method listenToRadio(frequency: Float)
        print "Tuning in the radio..."
        tuner.on()
        tuner.setFm()
        tuner.setFrequency(frequency)
        amp.on()
        amp.setTuner(tuner)
        amp.setStereoSound()
        amp.setVolume(5)
    end method

    method endRadio()
        print "Shutting down the tuner..."
        tuner.off()
        amp.off()
    end method

    method pauseMovie()
        player.pause()
        lights.dim(50)
    end method

    method resumeMovie()
        lights.dim(10)
        player.play(player.currentMovie)
    end method
end class

// Client code - dramatically simplified
theater = new HomeTheaterFacade(amp, tuner, player, projector, screen, lights, popper)

// One method call instead of many
theater.watchMovie("Inception")

// Later...
theater.pauseMovie()

// Resume
theater.resumeMovie()

// End the movie
theater.endMovie()

// Listen to radio - same simple interface
theater.listenToRadio(98.7)
theater.endRadio()
```

## Known Uses

- **SLF4J**: Simple Logging Facade for Java provides a unified logging interface over various logging frameworks (Log4j, Logback, java.util.logging).

- **jQuery**: Provides a simplified facade over the complex browser DOM APIs, normalizing differences between browsers.

- **Hibernate**: The `Session` class is a facade over the complex object-relational mapping subsystem.

- **Spring Framework**: `JdbcTemplate` provides a simplified facade over JDBC, handling connections, statements, and result sets.

- **AWS SDK**: High-level interfaces like `S3TransferManager` facade over low-level S3 operations.

- **React's `react-dom`**: The `render()` function is a facade over the complex reconciliation and DOM update process.

- **Compiler Front-ends**: A `compile()` method that facades lexing, parsing, semantic analysis, and code generation.

- **Payment Processors**: Libraries like Stripe's SDK facade over complex payment processing flows.

- **Database ORMs**: ActiveRecord, Eloquent, and similar ORMs facade over SQL and database connections.

## Related Patterns

- **Abstract Factory**: Can be used with Facade to provide an interface for creating subsystem objects in a subsystem-independent way. Abstract Factory can also be used as an alternative to Facade to hide platform-specific classes.

- **Mediator**: Similar to Facade in that it abstracts functionality of existing classes. Mediator's colleagues are aware of and communicate with the Mediator. Facade's subsystems are unaware of the Facade.

- **Singleton**: Facades are often Singletons because usually only one Facade object is required.

- **Adapter**: Adapter makes one interface conform to another. Facade defines a new interface over existing interfaces.

- **Flyweight**: Can be combined with Facade when the facade manages a pool of shared objects.

## When NOT to Use

- **When subsystem is already simple**: Adding a facade to an already simple subsystem adds unnecessary abstraction.

- **When clients need full control**: If most clients need access to the full power of the subsystem, a facade just becomes another interface to maintain.

- **As a replacement for good design**: Facades should not be used to hide poor subsystem design. Fix the underlying design instead.

- **When it becomes a god object**: If your facade grows to have dozens of methods, it is taking on too much responsibility. Consider multiple focused facades.

- **For mandatory access control**: Facade is an optional convenience. If you need to enforce access through a gateway, use a different pattern.

- **When behavior varies significantly**: If different clients need very different behaviors from the subsystem, a single facade may not serve them well.

- **In performance-critical paths**: The extra layer of indirection, while usually negligible, should be considered in tight loops.

---

## Summary

The Facade pattern provides a simple entry point to a complex subsystem. Its primary value is in reducing the apparent complexity of a system for clients who need common operations without requiring deep understanding of the subsystem. Key to successful use is keeping the facade focused - it should simplify common scenarios without becoming a god object. Remember that facades are optional conveniences, not gatekeepers; clients can always access the subsystem directly when needed. Use multiple focused facades for large subsystems rather than one monolithic facade.

---

*Based on concepts from "Design Patterns: Elements of Reusable Object-Oriented Software" by Gamma, Helm, Johnson, and Vlissides (Gang of Four), 1994.*
