# Decorator

## Intent

Attach additional responsibilities to an object dynamically. Decorators provide a flexible alternative to subclassing for extending functionality. The pattern allows behavior to be added to individual objects, either statically or dynamically, without affecting the behavior of other objects from the same class.

## Also Known As

- Wrapper

## Motivation

Sometimes you want to add responsibilities to individual objects, not to an entire class. A graphical user interface toolkit, for example, should let you add properties like borders or scroll bars to any user interface component.

One way to add responsibilities is with inheritance. Inheriting a border from another class puts a border around every subclass instance. This is inflexible, however, because the choice of border is made statically. A client cannot control how and when to decorate the component with a border.

A more flexible approach is to enclose the component in another object that adds the border. The enclosing object is called a decorator. The decorator conforms to the interface of the component it decorates so that its presence is transparent to the component's clients. The decorator forwards requests to the component and may perform additional actions before or after forwarding.

Consider a text view component in a document editor. A `TextView` displays text in a window. By default, text views do not have scroll bars because you may not always need them. When you do need them, you can use a `ScrollDecorator` to add them. You might also want to add a thick black border around the text view using a `BorderDecorator`. You can compose decorators - adding scroll bars, then a border - creating a border around a scrolling text view.

The key insight is that decorators and the original component share a common interface. From the client's perspective, whether they have a plain `TextView` or a decorated one with scrolling and borders, they interact with it the same way.

## Applicability

Use the Decorator pattern when:

- You want to add responsibilities to individual objects dynamically and transparently, without affecting other objects.

- You want to add responsibilities that can be withdrawn later.

- Extension by subclassing is impractical. Sometimes a large number of independent extensions are possible and would produce an explosion of subclasses to support every combination. Or a class definition may be hidden or otherwise unavailable for subclassing.

- You need to add cross-cutting concerns (logging, caching, validation) without modifying existing classes.

- You want to compose behaviors at runtime based on configuration or user choices.

- You need to follow the Single Responsibility Principle by dividing functionality between classes with unique areas of concern.

## Structure

```
┌─────────────────────────┐
│    <<interface>>        │
│      Component          │
├─────────────────────────┤
│ + operation()           │
└─────────────────────────┘
            △
            │
    ┌───────┴───────┐
    │               │
┌───────────┐  ┌────────────────────────┐
│ Concrete  │  │      Decorator         │
│ Component │  ├────────────────────────┤
├───────────┤  │ - component: Component │
│+operation()│ ├────────────────────────┤
└───────────┘  │ + operation()          │
               │   component.operation()│
               └────────────────────────┘
                           △
                           │
               ┌───────────┴───────────┐
               │                       │
    ┌──────────────────┐    ┌──────────────────┐
    │ConcreteDecoratorA│    │ConcreteDecoratorB│
    ├──────────────────┤    ├──────────────────┤
    │ + addedState     │    │ + operation()    │
    ├──────────────────┤    │   super.operation│
    │ + operation()    │    │   addedBehavior()│
    │   super.operation│    │ + addedBehavior()│
    │   // use state   │    └──────────────────┘
    └──────────────────┘
```

### Decorator Wrapping Example

```
┌────────────────────────────────────────────────────────────────┐
│                      BorderDecorator                            │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    ScrollDecorator                        │  │
│  │  ┌────────────────────────────────────────────────────┐  │  │
│  │  │                                                    │  │  │
│  │  │                    TextView                        │  │  │
│  │  │                                                    │  │  │
│  │  └────────────────────────────────────────────────────┘  │  │
│  │                      ↑ scroll bars                        │  │
│  └──────────────────────────────────────────────────────────┘  │
│                          border                                 │
└────────────────────────────────────────────────────────────────┘

Client calls draw() on BorderDecorator:
  → BorderDecorator draws border
  → Calls wrapped.draw() (ScrollDecorator)
    → ScrollDecorator draws scroll bars
    → Calls wrapped.draw() (TextView)
      → TextView draws text
```

## Participants

- **Component**: Defines the interface for objects that can have responsibilities added to them dynamically.

- **ConcreteComponent**: Defines an object to which additional responsibilities can be attached. This is the core object being decorated.

- **Decorator**: Maintains a reference to a Component object and defines an interface that conforms to Component's interface. This is often an abstract class.

- **ConcreteDecorator**: Adds responsibilities to the component. Each concrete decorator adds one specific responsibility or behavior.

## Collaborations

1. The Decorator forwards requests to its Component object.

2. The Decorator may optionally perform additional operations before and/or after forwarding the request.

3. Multiple decorators can be stacked, each adding its own behavior while delegating to the next decorator or the core component.

4. The client interacts with decorators through the Component interface, unaware of the decoration chain.

## Consequences

### Benefits

- **Greater flexibility than static inheritance**: You can add and remove responsibilities at runtime by attaching and detaching decorators. Inheritance requires creating a new class for each combination.

- **Avoids feature-laden classes high up in the hierarchy**: Instead of trying to support all features in a complex customizable class, you can define a simple class and add functionality incrementally with decorators.

- **Single Responsibility Principle**: You can divide functionality between classes with unique areas of concern. Each decorator handles one specific responsibility.

- **Open/Closed Principle**: You can extend behavior without modifying existing code. New decorators can be added without changing the component or other decorators.

- **Composable behaviors**: Decorators can be combined in various ways to create different behavior combinations. Order of decoration can matter.

- **Transparent to clients**: Decorated objects can be used anywhere the original object is expected.

### Liabilities

- **Lots of small objects**: A design that uses Decorator often results in systems composed of many small objects that all look alike. While easy to customize, these systems can be hard to debug.

- **Decorator and component are not identical**: A decorated component is not identical to the component itself. From an object identity standpoint, a decorated component is a different object.

- **Order dependency**: The order in which decorators are applied can matter. `border(scroll(textView))` might behave differently than `scroll(border(textView))`.

- **Interface conformance**: All decorators must conform to the component interface, which can be limiting if you need to expose decorator-specific methods.

- **Complexity in understanding flow**: With many decorators, understanding the actual flow of a method call through the chain can be challenging.

## Implementation

### Implementation Considerations

1. **Interface conformance**: The decorator must conform to the component interface. In typed languages, this means implementing the same interface or extending the same base class.

2. **Omitting the abstract Decorator class**: If you only need one decorator, you can merge Decorator's responsibility with ConcreteDecorator.

3. **Keeping Component classes lightweight**: To ensure decorators can be composed freely, keep the Component interface simple. Move complex state to concrete components.

4. **Changing the skin vs the guts**: Decorator changes the "skin" (external behavior) of an object; Strategy changes the "guts" (internal algorithm).

### Basic Decorator Implementation

```
// Component interface
interface DataSource
    method writeData(data: String)
    method readData(): String
end interface

// Concrete Component
class FileDataSource implements DataSource
    private filename: String

    constructor(filename: String)
        this.filename = filename
    end constructor

    method writeData(data: String)
        file = openFile(filename, "write")
        file.write(data)
        file.close()
    end method

    method readData(): String
        file = openFile(filename, "read")
        data = file.readAll()
        file.close()
        return data
    end method
end class

// Base Decorator
abstract class DataSourceDecorator implements DataSource
    protected wrappee: DataSource

    constructor(source: DataSource)
        this.wrappee = source
    end constructor

    method writeData(data: String)
        wrappee.writeData(data)
    end method

    method readData(): String
        return wrappee.readData()
    end method
end class

// Concrete Decorator: Encryption
class EncryptionDecorator extends DataSourceDecorator
    private encryptionKey: String

    constructor(source: DataSource, key: String)
        super(source)
        this.encryptionKey = key
    end constructor

    method writeData(data: String)
        // Encrypt before writing
        encryptedData = encrypt(data, encryptionKey)
        super.writeData(encryptedData)
    end method

    method readData(): String
        // Decrypt after reading
        encryptedData = super.readData()
        return decrypt(encryptedData, encryptionKey)
    end method

    private method encrypt(data: String, key: String): String
        // AES encryption implementation
        cipher = createCipher("AES", key)
        return base64Encode(cipher.encrypt(data.getBytes()))
    end method

    private method decrypt(data: String, key: String): String
        cipher = createCipher("AES", key)
        return new String(cipher.decrypt(base64Decode(data)))
    end method
end class

// Concrete Decorator: Compression
class CompressionDecorator extends DataSourceDecorator
    private compressionLevel: Integer

    constructor(source: DataSource, level: Integer)
        super(source)
        this.compressionLevel = level
    end constructor

    method writeData(data: String)
        // Compress before writing
        compressedData = compress(data)
        super.writeData(compressedData)
    end method

    method readData(): String
        // Decompress after reading
        compressedData = super.readData()
        return decompress(compressedData)
    end method

    private method compress(data: String): String
        compressor = new GzipCompressor(compressionLevel)
        return base64Encode(compressor.compress(data.getBytes()))
    end method

    private method decompress(data: String): String
        decompressor = new GzipDecompressor()
        return new String(decompressor.decompress(base64Decode(data)))
    end method
end class

// Usage - decorators can be composed in any order
// Order matters: compression then encryption vs encryption then compression
source = new FileDataSource("secrets.dat")

// Add compression
source = new CompressionDecorator(source, 9)

// Add encryption on top
source = new EncryptionDecorator(source, "my-secret-key")

// Client code - unaware of decorations
source.writeData("Sensitive data that needs protection")
data = source.readData()
print data  // "Sensitive data that needs protection"
```

### Decorator with Additional Methods

```
// When decorators need to expose additional functionality
interface Beverage
    method getDescription(): String
    method getCost(): Decimal
end interface

class Espresso implements Beverage
    method getDescription(): String
        return "Espresso"
    end method

    method getCost(): Decimal
        return 1.99
    end method
end class

class HouseBlend implements Beverage
    method getDescription(): String
        return "House Blend Coffee"
    end method

    method getCost(): Decimal
        return 0.89
    end method
end class

// Base decorator
abstract class CondimentDecorator implements Beverage
    protected beverage: Beverage

    constructor(beverage: Beverage)
        this.beverage = beverage
    end constructor

    method getDescription(): String
        return beverage.getDescription()
    end method
end class

// Concrete decorators
class Milk extends CondimentDecorator
    constructor(beverage: Beverage)
        super(beverage)
    end constructor

    method getDescription(): String
        return beverage.getDescription() + ", Milk"
    end method

    method getCost(): Decimal
        return beverage.getCost() + 0.20
    end method
end class

class Mocha extends CondimentDecorator
    constructor(beverage: Beverage)
        super(beverage)
    end constructor

    method getDescription(): String
        return beverage.getDescription() + ", Mocha"
    end method

    method getCost(): Decimal
        return beverage.getCost() + 0.30
    end method
end class

class Whip extends CondimentDecorator
    constructor(beverage: Beverage)
        super(beverage)
    end constructor

    method getDescription(): String
        return beverage.getDescription() + ", Whip"
    end method

    method getCost(): Decimal
        return beverage.getCost() + 0.25
    end method
end class

class SoyMilk extends CondimentDecorator
    constructor(beverage: Beverage)
        super(beverage)
    end constructor

    method getDescription(): String
        return beverage.getDescription() + ", Soy"
    end method

    method getCost(): Decimal
        return beverage.getCost() + 0.35
    end method
end class

// Usage - build complex drinks by stacking decorators
order1 = new Espresso()
print order1.getDescription() + " $" + order1.getCost()
// "Espresso $1.99"

order2 = new HouseBlend()
order2 = new Mocha(order2)
order2 = new Mocha(order2)  // Double mocha
order2 = new Whip(order2)
print order2.getDescription() + " $" + order2.getCost()
// "House Blend Coffee, Mocha, Mocha, Whip $1.74"

order3 = new Espresso()
order3 = new SoyMilk(order3)
order3 = new Mocha(order3)
order3 = new Whip(order3)
print order3.getDescription() + " $" + order3.getCost()
// "Espresso, Soy, Mocha, Whip $2.89"
```

## Example

### HTTP Middleware Pipeline

A real-world example where HTTP request handling is enhanced with cross-cutting concerns like logging, authentication, rate limiting, and caching.

```
// Component interface - HTTP handler
interface HttpHandler
    method handle(request: HttpRequest): HttpResponse
end interface

// Concrete component - actual request handler
class ApiEndpoint implements HttpHandler
    private controller: Controller
    private action: String

    constructor(controller: Controller, action: String)
        this.controller = controller
        this.action = action
    end constructor

    method handle(request: HttpRequest): HttpResponse
        try
            result = controller.invoke(action, request)
            return HttpResponse.ok(result)
        catch error
            return HttpResponse.serverError(error.message)
        end try
    end method
end class

// Base middleware decorator
abstract class Middleware implements HttpHandler
    protected next: HttpHandler

    constructor(next: HttpHandler)
        this.next = next
    end constructor

    method handle(request: HttpRequest): HttpResponse
        return next.handle(request)
    end method
end class

// Concrete Decorator: Logging
class LoggingMiddleware extends Middleware
    private logger: Logger

    constructor(next: HttpHandler, logger: Logger)
        super(next)
        this.logger = logger
    end constructor

    method handle(request: HttpRequest): HttpResponse
        requestId = generateRequestId()
        startTime = currentTimeMillis()

        // Log incoming request
        logger.info("[{}] {} {} - Started", requestId, request.method, request.path)
        logger.debug("[{}] Headers: {}", requestId, request.headers)

        // Call next handler
        response = next.handle(request)

        // Log response
        duration = currentTimeMillis() - startTime
        logger.info("[{}] {} {} - {} ({}ms)",
            requestId, request.method, request.path, response.status, duration)

        // Add request ID to response headers for tracing
        response.headers.set("X-Request-ID", requestId)

        return response
    end method
end class

// Concrete Decorator: Authentication
class AuthenticationMiddleware extends Middleware
    private authService: AuthService
    private excludedPaths: List<String>

    constructor(next: HttpHandler, authService: AuthService)
        super(next)
        this.authService = authService
        this.excludedPaths = ["/login", "/register", "/health", "/public"]
    end constructor

    method handle(request: HttpRequest): HttpResponse
        // Skip auth for excluded paths
        if isExcluded(request.path)
            return next.handle(request)
        end if

        // Extract token from Authorization header
        authHeader = request.headers.get("Authorization")
        if authHeader == null or not authHeader.startsWith("Bearer ")
            return HttpResponse.unauthorized("Missing or invalid Authorization header")
        end if

        token = authHeader.substring(7)

        try
            // Validate token and get user
            user = authService.validateToken(token)

            // Add user to request context for downstream handlers
            request.context.set("user", user)
            request.context.set("userId", user.id)
            request.context.set("roles", user.roles)

            return next.handle(request)
        catch InvalidTokenException
            return HttpResponse.unauthorized("Invalid or expired token")
        catch error
            return HttpResponse.serverError("Authentication error")
        end try
    end method

    private method isExcluded(path: String): Boolean
        for each excluded in excludedPaths
            if path.startsWith(excluded)
                return true
            end if
        end for
        return false
    end method
end class

// Concrete Decorator: Rate Limiting
class RateLimitMiddleware extends Middleware
    private rateLimiter: RateLimiter
    private requestsPerMinute: Integer
    private keyExtractor: Function

    constructor(next: HttpHandler, requestsPerMinute: Integer)
        super(next)
        this.requestsPerMinute = requestsPerMinute
        this.rateLimiter = new SlidingWindowRateLimiter()
        this.keyExtractor = request -> extractClientKey(request)
    end constructor

    method handle(request: HttpRequest): HttpResponse
        // Get rate limit key (IP, user ID, API key, etc.)
        clientKey = keyExtractor(request)

        // Check rate limit
        if not rateLimiter.tryAcquire(clientKey, requestsPerMinute)
            remaining = rateLimiter.getRemaining(clientKey)
            resetTime = rateLimiter.getResetTime(clientKey)

            response = HttpResponse.tooManyRequests("Rate limit exceeded")
            response.headers.set("X-RateLimit-Limit", requestsPerMinute.toString())
            response.headers.set("X-RateLimit-Remaining", remaining.toString())
            response.headers.set("X-RateLimit-Reset", resetTime.toString())
            response.headers.set("Retry-After", (resetTime - currentTime()).toString())

            return response
        end if

        // Process request
        response = next.handle(request)

        // Add rate limit headers to successful responses too
        remaining = rateLimiter.getRemaining(clientKey)
        response.headers.set("X-RateLimit-Limit", requestsPerMinute.toString())
        response.headers.set("X-RateLimit-Remaining", remaining.toString())

        return response
    end method

    private method extractClientKey(request: HttpRequest): String
        // Prefer user ID if authenticated
        userId = request.context.get("userId")
        if userId != null
            return "user:" + userId
        end if

        // Fall back to IP address
        return "ip:" + request.getClientIP()
    end method
end class

// Concrete Decorator: Caching
class CacheMiddleware extends Middleware
    private cache: Cache
    private defaultTTL: Duration
    private cacheableMethod: Set<String>

    constructor(next: HttpHandler, cache: Cache)
        super(next)
        this.cache = cache
        this.defaultTTL = Duration.minutes(5)
        this.cacheableMethod = Set.of("GET", "HEAD")
    end constructor

    method handle(request: HttpRequest): HttpResponse
        // Only cache GET and HEAD requests
        if not cacheableMethod.contains(request.method)
            return next.handle(request)
        end if

        // Check for no-cache directive
        if request.headers.get("Cache-Control") == "no-cache"
            return next.handle(request)
        end if

        // Generate cache key
        cacheKey = generateCacheKey(request)

        // Try to get from cache
        cachedResponse = cache.get(cacheKey)
        if cachedResponse != null
            cachedResponse.headers.set("X-Cache", "HIT")
            return cachedResponse
        end if

        // Cache miss - call next handler
        response = next.handle(request)

        // Only cache successful responses
        if response.status >= 200 and response.status < 300
            ttl = parseCacheControl(response) or defaultTTL
            cache.set(cacheKey, response, ttl)
            response.headers.set("X-Cache", "MISS")
        end if

        return response
    end method

    private method generateCacheKey(request: HttpRequest): String
        // Include relevant headers in cache key
        varyHeaders = request.headers.get("Vary") or ""
        return hash(request.method + ":" + request.path + ":" + request.query + ":" + varyHeaders)
    end method

    private method parseCacheControl(response: HttpResponse): Duration
        cacheControl = response.headers.get("Cache-Control")
        if cacheControl == null
            return null
        end if

        // Parse max-age directive
        match = regex("max-age=(\d+)").match(cacheControl)
        if match != null
            return Duration.seconds(parseInt(match.group(1)))
        end if

        return null
    end method
end class

// Concrete Decorator: Error Handling
class ErrorHandlingMiddleware extends Middleware
    private errorReporter: ErrorReporter
    private showStackTrace: Boolean

    constructor(next: HttpHandler, errorReporter: ErrorReporter)
        super(next)
        this.errorReporter = errorReporter
        this.showStackTrace = Environment.isDevelopment()
    end constructor

    method handle(request: HttpRequest): HttpResponse
        try
            return next.handle(request)
        catch ValidationException as e
            return HttpResponse.badRequest({
                error: "Validation Error",
                message: e.message,
                fields: e.fieldErrors
            })
        catch NotFoundException as e
            return HttpResponse.notFound({
                error: "Not Found",
                message: e.message
            })
        catch ForbiddenException as e
            return HttpResponse.forbidden({
                error: "Forbidden",
                message: e.message
            })
        catch error
            // Report unexpected errors
            errorId = errorReporter.report(error, {
                request: request,
                user: request.context.get("user")
            })

            body = {
                error: "Internal Server Error",
                errorId: errorId,
                message: showStackTrace ? error.message : "An unexpected error occurred"
            }

            if showStackTrace
                body.stackTrace = error.stackTrace
            end if

            return HttpResponse.serverError(body)
        end try
    end method
end class

// Concrete Decorator: CORS
class CorsMiddleware extends Middleware
    private allowedOrigins: Set<String>
    private allowedMethods: Set<String>
    private allowedHeaders: Set<String>
    private maxAge: Integer

    constructor(next: HttpHandler, config: CorsConfig)
        super(next)
        this.allowedOrigins = config.origins
        this.allowedMethods = config.methods
        this.allowedHeaders = config.headers
        this.maxAge = config.maxAge
    end constructor

    method handle(request: HttpRequest): HttpResponse
        origin = request.headers.get("Origin")

        // Handle preflight requests
        if request.method == "OPTIONS"
            return handlePreflight(request, origin)
        end if

        // Process actual request
        response = next.handle(request)

        // Add CORS headers if origin is allowed
        if isOriginAllowed(origin)
            response.headers.set("Access-Control-Allow-Origin", origin)
            response.headers.set("Access-Control-Allow-Credentials", "true")
        end if

        return response
    end method

    private method handlePreflight(request: HttpRequest, origin: String): HttpResponse
        if not isOriginAllowed(origin)
            return HttpResponse.forbidden("Origin not allowed")
        end if

        response = HttpResponse.noContent()
        response.headers.set("Access-Control-Allow-Origin", origin)
        response.headers.set("Access-Control-Allow-Methods", allowedMethods.join(", "))
        response.headers.set("Access-Control-Allow-Headers", allowedHeaders.join(", "))
        response.headers.set("Access-Control-Max-Age", maxAge.toString())
        response.headers.set("Access-Control-Allow-Credentials", "true")

        return response
    end method

    private method isOriginAllowed(origin: String): Boolean
        if origin == null
            return false
        end if
        return allowedOrigins.contains("*") or allowedOrigins.contains(origin)
    end method
end class

// Concrete Decorator: Request Validation
class ValidationMiddleware extends Middleware
    private validator: RequestValidator
    private schemas: Map<String, Schema>

    constructor(next: HttpHandler, schemas: Map<String, Schema>)
        super(next)
        this.validator = new RequestValidator()
        this.schemas = schemas
    end constructor

    method handle(request: HttpRequest): HttpResponse
        // Find schema for this endpoint
        schemaKey = request.method + ":" + request.path
        schema = schemas.get(schemaKey)

        if schema != null
            // Validate request body
            if schema.body != null and request.body != null
                errors = validator.validate(request.body, schema.body)
                if not errors.isEmpty()
                    return HttpResponse.badRequest({
                        error: "Validation Error",
                        errors: errors
                    })
                end if
            end if

            // Validate query parameters
            if schema.query != null
                errors = validator.validate(request.query, schema.query)
                if not errors.isEmpty()
                    return HttpResponse.badRequest({
                        error: "Invalid Query Parameters",
                        errors: errors
                    })
                end if
            end if
        end if

        return next.handle(request)
    end method
end class

// Building the middleware pipeline
class Application
    method createHandler(): HttpHandler
        // Create the actual endpoint handler
        handler = new ApiEndpoint(userController, "getUser")

        // Wrap with middleware (order matters!)
        // Error handling is outermost to catch all errors
        handler = new ErrorHandlingMiddleware(handler, errorReporter)

        // CORS before authentication
        handler = new CorsMiddleware(handler, corsConfig)

        // Logging to trace all requests
        handler = new LoggingMiddleware(handler, logger)

        // Rate limiting before expensive operations
        handler = new RateLimitMiddleware(handler, 100)

        // Authentication
        handler = new AuthenticationMiddleware(handler, authService)

        // Validation
        handler = new ValidationMiddleware(handler, schemas)

        // Caching closest to the endpoint
        handler = new CacheMiddleware(handler, cache)

        return handler
    end method
end class

// Usage - client code is simple
server = new HttpServer()
handler = app.createHandler()

server.on("request", (request) -> {
    response = handler.handle(request)
    server.send(response)
})

// Middleware can also be configured per-route
router = new Router()

// Public routes - no auth
publicHandler = new LoggingMiddleware(
    new RateLimitMiddleware(
        new ApiEndpoint(publicController, "index"),
        1000
    ),
    logger
)
router.get("/public", publicHandler)

// Admin routes - extra security
adminHandler = new LoggingMiddleware(
    new AuthenticationMiddleware(
        new RateLimitMiddleware(
            new ApiEndpoint(adminController, "dashboard"),
            10
        ),
        authService
    ),
    logger
)
router.get("/admin", adminHandler)
```

## Known Uses

- **Java I/O Streams**: `BufferedInputStream`, `DataInputStream`, `GZIPInputStream` are decorators for `InputStream`. You can compose them: `new BufferedInputStream(new GZIPInputStream(new FileInputStream(file)))`.

- **Python functools**: The `@functools.wraps` decorator and custom function decorators use this pattern to add behavior to functions.

- **Express.js/Koa Middleware**: HTTP middleware in Node.js frameworks use decorator-like patterns for logging, authentication, parsing, etc.

- **Spring Framework**: Aspects and interceptors wrap service methods to add transaction management, security, and logging.

- **React Higher-Order Components (HOCs)**: Functions like `withRouter`, `connect`, and `memo` wrap components to add functionality.

- **Django Middleware**: Request/response processing pipeline where each middleware can modify requests before they reach views.

- **Java Collections**: `Collections.synchronizedList()`, `Collections.unmodifiableList()` return decorated lists with added behavior.

- **Graphics Libraries**: Border decorators, scroll decorators in GUI toolkits (Java Swing's `JScrollPane`).

- **Logging Facades**: SLF4J's MDC (Mapped Diagnostic Context) decorates log messages with context.

## Related Patterns

- **Adapter**: Changes an object's interface; Decorator enhances without changing interface.

- **Composite**: Decorator can be viewed as a degenerate Composite with only one component. Decorator adds responsibilities; Composite aggregates objects.

- **Strategy**: Decorator changes the skin of an object; Strategy changes the guts. With Strategy, the component knows about extensions; with Decorator, it does not.

- **Proxy**: Both patterns compose objects and forward requests. Proxy controls access; Decorator adds responsibilities.

- **Chain of Responsibility**: Similar structure with forwarding. Chain passes requests until one handles it; Decorator always forwards.

## When NOT to Use

- **When subclassing is sufficient**: If you have a small, fixed set of variations and do not need runtime composition, simple inheritance may be clearer.

- **When the component interface is large**: Decorators must implement the entire component interface. With many methods, most of which just delegate, decorator classes become bloated.

- **When order independence is required**: If the result should not depend on the order of applying decorations, Decorator may lead to subtle bugs.

- **When identity matters**: If clients check object identity (reference equality), decorated objects will fail those checks.

- **For performance-critical paths**: Each decorator adds a method call. In tight loops, the overhead may be measurable.

- **When state needs to be shared**: Decorators work best with stateless enhancements. If decorators need to share state, the design becomes complicated.

- **Simple cross-cutting concerns**: For truly simple cases like logging a single method, aspect-oriented programming or simple method wrappers may be cleaner than full Decorator pattern.

- **When behavior modification is complex**: If the "enhancement" significantly changes the semantics of operations rather than just wrapping them, a different pattern may be more appropriate.

---

## Summary

The Decorator pattern provides an elegant way to add responsibilities to objects dynamically. Its key strengths are runtime flexibility and adherence to the Single Responsibility Principle - each decorator handles one concern. The pattern is especially powerful for cross-cutting concerns like logging, caching, and authentication. However, it can lead to many small classes and debugging complexity when overused. Choose Decorator when you need to compose behaviors flexibly; prefer simpler approaches when the set of variations is small and fixed.

---

*Based on concepts from "Design Patterns: Elements of Reusable Object-Oriented Software" by Gamma, Helm, Johnson, and Vlissides (Gang of Four), 1994.*
