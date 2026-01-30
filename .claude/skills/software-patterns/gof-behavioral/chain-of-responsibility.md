# Chain of Responsibility

## Intent

Avoid coupling the sender of a request to its receiver by giving more than one object a chance to handle the request. Chain the receiving objects and pass the request along the chain until an object handles it.

## Also Known As

- CoR
- Chain of Command
- Responsibility Chain

## Motivation

Consider a context-sensitive help system for a graphical user interface. When a user requests help, the system should provide information based on the context—the specific UI element the user is interacting with. However, not every UI element has its own help documentation. A button inside a dialog inside a panel might not have specific help, but its containing dialog might, or ultimately the application itself might provide generic help.

The challenge is that the object initiating the help request (the button) doesn't know which object will ultimately handle it. We need a way to decouple the sender from the receiver while still ensuring the request gets handled appropriately.

The Chain of Responsibility pattern addresses this by creating a chain of handler objects. Each handler has a reference to its successor in the chain. When a request comes in, each handler decides either to process it or pass it to the next handler. The button passes the help request to its parent dialog, which either handles it or passes it to the panel, which passes it to the application window. This continues until something handles the request or the chain ends.

This decoupling is powerful because the sender doesn't need to know the chain's structure. You can add, remove, or reorder handlers without changing the requesting code. The button doesn't care whether the dialog, panel, or application ultimately provides help—it simply initiates the request.

## Applicability

Use the Chain of Responsibility pattern when:

- More than one object may handle a request, and the handler isn't known a priori. The handler should be ascertained automatically.
- You want to issue a request to one of several objects without specifying the receiver explicitly.
- The set of objects that can handle a request should be specified dynamically.
- You need to process a request through multiple handlers in sequence (like middleware).
- The order of handler execution matters and should be configurable.
- You want to decouple request senders from receivers.
- You have a hierarchy of objects where requests naturally flow from specific to general.

## Structure

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│    ┌────────┐         ┌───────────────────────────────────────────┐    │
│    │ Client │────────>│              <<interface>>                │    │
│    └────────┘         │                 Handler                   │    │
│                       ├───────────────────────────────────────────┤    │
│                       │ + setNext(handler: Handler): Handler      │    │
│                       │ + handle(request: Request): Response      │    │
│                       └───────────────────────────────────────────┘    │
│                                          △                             │
│                                          │                             │
│                    ┌─────────────────────┼─────────────────────┐       │
│                    │                     │                     │       │
│         ┌──────────┴──────────┐ ┌───────┴────────┐ ┌─────────┴──────┐ │
│         │   BaseHandler       │ │ ConcreteHandlerA│ │ConcreteHandlerB│ │
│         ├─────────────────────┤ ├─────────────────┤ ├────────────────┤ │
│         │ - next: Handler     │ │                 │ │                │ │
│         ├─────────────────────┤ ├─────────────────┤ ├────────────────┤ │
│         │ + setNext(handler)  │ │ + handle(req)   │ │ + handle(req)  │ │
│         │ + handle(request)   │ │   // process or │ │   // process or│ │
│         │   // pass to next   │ │   // pass along │ │   // pass along│ │
│         └─────────────────────┘ └─────────────────┘ └────────────────┘ │
│                                                                         │
│    Request Flow:                                                        │
│    ┌──────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐           │
│    │Client│───>│Handler A │───>│Handler B │───>│Handler C │           │
│    └──────┘    └──────────┘    └──────────┘    └──────────┘           │
│                     │               │               │                  │
│                  handles?        handles?        handles?              │
│                   No->             No->            Yes!                │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## Participants

- **Handler**: Declares the interface for handling requests. Optionally implements the successor link.

- **BaseHandler** (optional): Implements default handling behavior, which is to forward the request to the successor. Concrete handlers can extend this to inherit the forwarding logic.

- **ConcreteHandler**: Handles requests it is responsible for. Can access its successor. If the ConcreteHandler can handle the request, it does so; otherwise, it forwards the request to its successor.

- **Client**: Initiates the request to a ConcreteHandler object on the chain. The client doesn't know which handler will ultimately process the request.

## Collaborations

1. When a client issues a request, the request propagates along the chain until a ConcreteHandler object takes responsibility for handling it.

2. The client that initiates the request has no direct knowledge of which object will ultimately handle it. We say the request has an implicit receiver.

3. Each handler in the chain either handles the request or passes it to the next handler. Some implementations allow a handler to do both—process the request and pass it along for additional processing.

4. If the request reaches the end of the chain without being handled, implementations typically either ignore it, throw an exception, or have a default handler that always processes requests.

## Consequences

### Benefits

1. **Reduced coupling**: The pattern frees an object from knowing which other object handles a request. An object only has to know that a request will be handled "appropriately." Both the receiver and the sender have no explicit knowledge of each other, and an object in the chain doesn't have to know about the chain's structure.

2. **Added flexibility in assigning responsibilities to objects**: You can add or change responsibilities for handling a request by adding to or changing the chain at runtime. You can combine this with subclassing to specialize handlers statically.

3. **Single Responsibility Principle**: You can decouple classes that invoke operations from classes that perform operations.

4. **Open/Closed Principle**: You can introduce new handlers into the application without breaking existing client code.

5. **Control over order**: The chain can be configured to process requests in a specific order, and this order can be changed at runtime.

### Liabilities

1. **Receipt isn't guaranteed**: Since a request has no explicit receiver, there's no guarantee it will be handled. The request can fall off the end of the chain without ever being handled. A request can also go unhandled when the chain is not configured properly.

2. **Runtime configuration complexity**: Debugging can be challenging because the request processing flows through multiple objects. You need to trace through the chain to understand the runtime flow.

3. **Performance overhead**: If the chain is long, there's overhead in traversing from handler to handler. Each handler might also perform its own checks before deciding to handle or pass.

4. **Potential infinite loops**: If the chain is circular (accidentally or by design), requests can loop forever without being handled.

## Implementation

### Implementation Considerations

1. **Implementing the successor chain**: There are two approaches:
   - Define new links (usually in the Handler itself)
   - Use existing links (like parent references in a composite structure)

2. **Connecting successors**: If there are no preexisting references to define a chain, you'll have to introduce them. The Handler defines the interface for handling requests and for accessing the successor.

3. **Representing requests**: There are several ways to represent requests:
   - A single hardcoded operation (inflexible but type-safe)
   - A request object that encapsulates all parameters
   - A request code or type with separate parameters

### Pseudocode: Implementing the Chain

```
// Define the handler interface
interface SupportHandler {
    method setNext(handler: SupportHandler): SupportHandler
    method handle(ticket: SupportTicket): string
}

// Abstract base handler with default forwarding behavior
abstract class BaseSupportHandler implements SupportHandler {
    private nextHandler: SupportHandler = null

    method setNext(handler: SupportHandler): SupportHandler {
        this.nextHandler = handler
        // Return handler to allow chaining: h1.setNext(h2).setNext(h3)
        return handler
    }

    method handle(ticket: SupportTicket): string {
        if this.nextHandler is not null {
            return this.nextHandler.handle(ticket)
        }
        return "No handler could process the ticket"
    }
}

// Concrete handler for basic support issues
class BasicSupportHandler extends BaseSupportHandler {
    method handle(ticket: SupportTicket): string {
        if ticket.severity == "LOW" and ticket.category == "FAQ" {
            return "BasicSupport: Resolved with FAQ article #" + findFaqArticle(ticket)
        }
        // Pass to next handler
        return super.handle(ticket)
    }

    private method findFaqArticle(ticket: SupportTicket): string {
        // Look up relevant FAQ
        return "KB-" + hash(ticket.description)
    }
}

// Concrete handler for technical issues
class TechnicalSupportHandler extends BaseSupportHandler {
    method handle(ticket: SupportTicket): string {
        if ticket.category == "TECHNICAL" and ticket.severity in ["LOW", "MEDIUM"] {
            return "TechSupport: Assigned to technician, ETA 24 hours"
        }
        return super.handle(ticket)
    }
}

// Concrete handler for billing issues
class BillingSupportHandler extends BaseSupportHandler {
    method handle(ticket: SupportTicket): string {
        if ticket.category == "BILLING" {
            return "BillingSupport: Forwarded to finance team, ETA 48 hours"
        }
        return super.handle(ticket)
    }
}

// Concrete handler for escalated issues
class EscalationHandler extends BaseSupportHandler {
    method handle(ticket: SupportTicket): string {
        if ticket.severity == "CRITICAL" or ticket.escalated {
            return "Escalation: Priority ticket created, manager notified"
        }
        return super.handle(ticket)
    }
}

// Default handler that catches everything else
class DefaultHandler extends BaseSupportHandler {
    method handle(ticket: SupportTicket): string {
        // This is the end of the chain - handle everything that falls through
        return "DefaultHandler: Ticket queued for manual review"
    }
}
```

### Pseudocode: Request Object

```
class SupportTicket {
    public id: string
    public customerId: string
    public category: string      // "FAQ", "TECHNICAL", "BILLING", "OTHER"
    public severity: string      // "LOW", "MEDIUM", "HIGH", "CRITICAL"
    public description: string
    public escalated: boolean
    public createdAt: timestamp
    public metadata: map<string, any>

    constructor(category: string, severity: string, description: string) {
        this.id = generateUUID()
        this.category = category
        this.severity = severity
        this.description = description
        this.escalated = false
        this.createdAt = now()
        this.metadata = {}
    }

    method escalate(): void {
        this.escalated = true
        this.severity = "CRITICAL"
    }
}
```

### Pseudocode: Chain Assembly

```
class SupportSystem {
    private handlerChain: SupportHandler

    constructor() {
        // Build the chain
        // Order matters: more specific handlers first, catch-all last
        basic = new BasicSupportHandler()
        technical = new TechnicalSupportHandler()
        billing = new BillingSupportHandler()
        escalation = new EscalationHandler()
        default = new DefaultHandler()

        // Chain them together
        basic.setNext(technical)
             .setNext(billing)
             .setNext(escalation)
             .setNext(default)

        this.handlerChain = basic
    }

    method submitTicket(ticket: SupportTicket): string {
        log("Processing ticket: " + ticket.id)
        result = this.handlerChain.handle(ticket)
        log("Result: " + result)
        return result
    }

    // Allows runtime reconfiguration
    method setHandlerChain(firstHandler: SupportHandler): void {
        this.handlerChain = firstHandler
    }
}
```

## Example

A complete example implementing HTTP middleware for a web application:

```
// Request object representing an HTTP request
class HttpRequest {
    public method: string
    public path: string
    public headers: map<string, string>
    public body: any
    public user: User = null
    public startTime: timestamp
    public attributes: map<string, any> = {}

    constructor(method: string, path: string, headers: map, body: any) {
        this.method = method
        this.path = path
        this.headers = headers
        this.body = body
        this.startTime = now()
    }

    method getAttribute(key: string): any {
        return this.attributes.get(key)
    }

    method setAttribute(key: string, value: any): void {
        this.attributes.set(key, value)
    }
}

// Response object
class HttpResponse {
    public statusCode: int = 200
    public headers: map<string, string> = {}
    public body: any = null

    method setStatus(code: int): HttpResponse {
        this.statusCode = code
        return this
    }

    method setHeader(name: string, value: string): HttpResponse {
        this.headers.set(name, value)
        return this
    }

    method setBody(body: any): HttpResponse {
        this.body = body
        return this
    }
}

// Middleware interface (Handler)
interface Middleware {
    method setNext(middleware: Middleware): Middleware
    method handle(request: HttpRequest, response: HttpResponse): HttpResponse
}

// Base middleware with chain forwarding
abstract class BaseMiddleware implements Middleware {
    private next: Middleware = null

    method setNext(middleware: Middleware): Middleware {
        this.next = middleware
        return middleware
    }

    method handle(request: HttpRequest, response: HttpResponse): HttpResponse {
        if this.next is not null {
            return this.next.handle(request, response)
        }
        return response
    }

    protected method passToNext(request: HttpRequest, response: HttpResponse): HttpResponse {
        return this.handle(request, response)
    }
}

// Logging middleware - logs all requests (processes AND passes)
class LoggingMiddleware extends BaseMiddleware {
    private logger: Logger

    constructor(logger: Logger) {
        this.logger = logger
    }

    method handle(request: HttpRequest, response: HttpResponse): HttpResponse {
        // Log incoming request
        this.logger.info("Incoming: " + request.method + " " + request.path)

        // Pass to next handler
        response = super.handle(request, response)

        // Log response (after chain completes)
        duration = now() - request.startTime
        this.logger.info("Response: " + response.statusCode + " in " + duration + "ms")

        return response
    }
}

// Authentication middleware - blocks unauthorized requests
class AuthenticationMiddleware extends BaseMiddleware {
    private authService: AuthService
    private publicPaths: list<string>

    constructor(authService: AuthService) {
        this.authService = authService
        this.publicPaths = ["/login", "/register", "/health", "/public/*"]
    }

    method handle(request: HttpRequest, response: HttpResponse): HttpResponse {
        // Skip auth for public paths
        if this.isPublicPath(request.path) {
            return super.handle(request, response)
        }

        // Check for auth token
        token = request.headers.get("Authorization")
        if token is null or token.isEmpty() {
            return response.setStatus(401).setBody({"error": "Missing authentication"})
        }

        // Validate token
        user = this.authService.validateToken(token)
        if user is null {
            return response.setStatus(401).setBody({"error": "Invalid token"})
        }

        // Attach user to request and continue
        request.user = user
        return super.handle(request, response)
    }

    private method isPublicPath(path: string): boolean {
        for publicPath in this.publicPaths {
            if publicPath.endsWith("/*") {
                prefix = publicPath.substring(0, publicPath.length - 2)
                if path.startsWith(prefix) {
                    return true
                }
            } else if path == publicPath {
                return true
            }
        }
        return false
    }
}

// Rate limiting middleware
class RateLimitMiddleware extends BaseMiddleware {
    private rateLimiter: RateLimiter
    private requestsPerMinute: int

    constructor(requestsPerMinute: int) {
        this.requestsPerMinute = requestsPerMinute
        this.rateLimiter = new SlidingWindowRateLimiter(requestsPerMinute)
    }

    method handle(request: HttpRequest, response: HttpResponse): HttpResponse {
        // Identify client (by IP or user ID)
        clientId = this.getClientIdentifier(request)

        if not this.rateLimiter.allowRequest(clientId) {
            retryAfter = this.rateLimiter.getRetryAfter(clientId)
            return response
                .setStatus(429)
                .setHeader("Retry-After", retryAfter.toString())
                .setBody({"error": "Rate limit exceeded", "retryAfter": retryAfter})
        }

        // Add rate limit headers
        remaining = this.rateLimiter.getRemainingRequests(clientId)
        response.setHeader("X-RateLimit-Limit", this.requestsPerMinute.toString())
        response.setHeader("X-RateLimit-Remaining", remaining.toString())

        return super.handle(request, response)
    }

    private method getClientIdentifier(request: HttpRequest): string {
        if request.user is not null {
            return "user:" + request.user.id
        }
        return "ip:" + request.headers.get("X-Forwarded-For", "unknown")
    }
}

// CORS middleware
class CorsMiddleware extends BaseMiddleware {
    private allowedOrigins: list<string>
    private allowedMethods: list<string>

    constructor(allowedOrigins: list<string>) {
        this.allowedOrigins = allowedOrigins
        this.allowedMethods = ["GET", "POST", "PUT", "DELETE", "OPTIONS"]
    }

    method handle(request: HttpRequest, response: HttpResponse): HttpResponse {
        origin = request.headers.get("Origin")

        if origin is not null and this.isAllowedOrigin(origin) {
            response.setHeader("Access-Control-Allow-Origin", origin)
            response.setHeader("Access-Control-Allow-Methods", join(this.allowedMethods, ", "))
            response.setHeader("Access-Control-Allow-Headers", "Content-Type, Authorization")
            response.setHeader("Access-Control-Max-Age", "86400")
        }

        // Handle preflight requests
        if request.method == "OPTIONS" {
            return response.setStatus(204)
        }

        return super.handle(request, response)
    }

    private method isAllowedOrigin(origin: string): boolean {
        return "*" in this.allowedOrigins or origin in this.allowedOrigins
    }
}

// Request validation middleware
class ValidationMiddleware extends BaseMiddleware {
    private validators: map<string, Validator>

    constructor() {
        this.validators = {}
    }

    method registerValidator(pathPattern: string, validator: Validator): void {
        this.validators.set(pathPattern, validator)
    }

    method handle(request: HttpRequest, response: HttpResponse): HttpResponse {
        validator = this.findValidator(request.path)

        if validator is not null {
            errors = validator.validate(request)
            if errors.isNotEmpty() {
                return response.setStatus(400).setBody({
                    "error": "Validation failed",
                    "details": errors
                })
            }
        }

        return super.handle(request, response)
    }

    private method findValidator(path: string): Validator {
        for pattern, validator in this.validators {
            if pathMatches(path, pattern) {
                return validator
            }
        }
        return null
    }
}

// Final handler - the actual route processor
class RouteHandler extends BaseMiddleware {
    private router: Router

    constructor(router: Router) {
        this.router = router
    }

    method handle(request: HttpRequest, response: HttpResponse): HttpResponse {
        handler = this.router.findHandler(request.method, request.path)

        if handler is null {
            return response.setStatus(404).setBody({"error": "Not found"})
        }

        try {
            return handler.execute(request, response)
        } catch Exception as e {
            return response.setStatus(500).setBody({
                "error": "Internal server error",
                "message": e.getMessage()
            })
        }
    }
}

// Application setup
class WebApplication {
    private middlewareChain: Middleware
    private router: Router

    constructor() {
        this.router = new Router()
        this.setupMiddleware()
    }

    private method setupMiddleware(): void {
        // Create middleware instances
        logging = new LoggingMiddleware(new ConsoleLogger())
        cors = new CorsMiddleware(["https://example.com", "https://app.example.com"])
        rateLimit = new RateLimitMiddleware(100)  // 100 requests per minute
        auth = new AuthenticationMiddleware(new JwtAuthService())
        validation = new ValidationMiddleware()
        routes = new RouteHandler(this.router)

        // Build the chain
        // Order: logging -> cors -> rateLimit -> auth -> validation -> routes
        logging.setNext(cors)
                .setNext(rateLimit)
                .setNext(auth)
                .setNext(validation)
                .setNext(routes)

        this.middlewareChain = logging
    }

    method handleRequest(request: HttpRequest): HttpResponse {
        response = new HttpResponse()
        return this.middlewareChain.handle(request, response)
    }

    method addRoute(method: string, path: string, handler: RouteHandlerFunc): void {
        this.router.addRoute(method, path, handler)
    }
}

// Usage example
function main() {
    app = new WebApplication()

    // Register routes
    app.addRoute("GET", "/api/users", getUsersHandler)
    app.addRoute("POST", "/api/users", createUserHandler)
    app.addRoute("GET", "/health", healthCheckHandler)

    // Simulate incoming requests
    request1 = new HttpRequest("GET", "/api/users", {"Authorization": "Bearer valid-token"}, null)
    response1 = app.handleRequest(request1)
    print(response1.statusCode)  // 200

    request2 = new HttpRequest("GET", "/api/users", {}, null)
    response2 = app.handleRequest(request2)
    print(response2.statusCode)  // 401 - no auth token

    request3 = new HttpRequest("GET", "/health", {}, null)
    response3 = app.handleRequest(request3)
    print(response3.statusCode)  // 200 - public path, no auth needed
}
```

## Known Uses

- **Java Servlet Filters**: The `javax.servlet.Filter` interface implements Chain of Responsibility for processing HTTP requests. Filters can examine/modify requests and responses or block further processing.

- **Express.js/Connect Middleware**: Node.js web frameworks use middleware functions that receive `(req, res, next)` and call `next()` to pass control to the next middleware.

- **Django Middleware**: Python's Django framework processes requests through a configurable chain of middleware classes.

- **Apache Commons Chain**: A reusable implementation of the Chain of Responsibility pattern for building command processing pipelines.

- **Spring Security Filter Chain**: Authentication and authorization are processed through a chain of security filters.

- **DOM Event Bubbling**: In web browsers, events bubble up through the DOM tree, each element getting a chance to handle the event.

- **ATM Dispensing**: ATMs use this pattern to dispense bills—$100 handler tries first, then $50, then $20, etc.

- **Exception Handling**: Many languages implement exception handling as a chain where each catch block decides whether to handle or rethrow.

- **Logging Frameworks (Log4j, SLF4J)**: Log levels form a chain where DEBUG handlers might pass to INFO, which passes to WARN, etc.

## Related Patterns

- **Composite**: Chain of Responsibility is often applied in conjunction with Composite. A component's parent can act as its successor in the chain.

- **Command**: Commands can be placed in a queue and processed by a chain of handlers, combining both patterns for sophisticated request processing.

- **Decorator**: Both patterns rely on recursive composition. However, Decorator adds responsibilities without changing the interface, while Chain of Responsibility can change the flow of execution based on which handler processes the request.

- **Observer**: Chain of Responsibility passes a request along a chain until handled; Observer broadcasts to all interested observers. Chain of Responsibility is sequential; Observer is parallel.

## When NOT to Use

1. **When you need guaranteed handling**: If every request MUST be processed, Chain of Responsibility's implicit receiver makes it harder to ensure this. Use a direct call or Strategy pattern instead.

2. **When order doesn't matter**: If handlers are independent and order is irrelevant, consider Observer or a simple list of handlers that all process the request.

3. **When performance is critical**: The overhead of traversing a long chain for every request may be unacceptable. Consider caching handler decisions or using a lookup table.

4. **When debugging transparency is essential**: In highly regulated environments where you need clear audit trails of exactly which code processed a request, the indirection of Chain of Responsibility may be inappropriate.

5. **For simple conditional logic**: If you have 2-3 conditions, a simple if-else or switch statement is clearer and more maintainable than setting up a chain infrastructure.

6. **When handlers need to know about each other**: The pattern assumes handlers are independent. If handler B's behavior depends on whether handler A processed the request, you're fighting the pattern.

7. **When request types are fixed and known at compile time**: If all request types are known and won't change, a more direct dispatch mechanism (like polymorphism or a type-based lookup) may be simpler.

```
// Anti-pattern: Using Chain of Responsibility for two simple conditions
// DON'T DO THIS:
basicHandler.setNext(premiumHandler)
                   .setNext(defaultHandler)

// Just use if-else instead:
if user.isPremium() {
    return handlePremium(request)
} else if user.isBasic() {
    return handleBasic(request)
} else {
    return handleDefault(request)
}
```

The Chain of Responsibility pattern shines when you need flexibility, extensibility, and decoupling. Use it for middleware pipelines, event processing, and scenarios where the handling logic may change at runtime. Avoid it when simplicity and directness are more valuable.

---

*Based on concepts from "Design Patterns: Elements of Reusable Object-Oriented Software" by Gamma, Helm, Johnson, and Vlissides (Gang of Four), 1994.*
