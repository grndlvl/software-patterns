# Builder

## Intent

Separate the construction of a complex object from its representation so that the same construction process can create different representations. The Builder pattern allows you to produce different types and representations of an object using the same construction code.

## Also Known As

- Step Builder
- Fluent Builder

## Motivation

Consider an application that needs to create SQL queries. A query can be simple or complex: it might have SELECT clauses with columns and aliases, WHERE conditions with various operators, JOIN clauses linking multiple tables, GROUP BY with HAVING conditions, ORDER BY with multiple columns and directions, and LIMIT/OFFSET for pagination. Creating such queries by concatenating strings is error-prone, hard to read, and vulnerable to SQL injection.

We could define a Query class with a constructor that takes all possible parameters, but this leads to telescoping constructors (constructors with many parameters, most of which are optional). The constructor call becomes unreadable, and it's easy to mix up parameter order.

The Builder pattern solves this by separating the construction of the Query object from its representation. A QueryBuilder class provides methods to configure each part of the query step by step. Each method returns the builder itself, enabling method chaining. When configuration is complete, a build() method creates the final Query object. The same building process can create different representations—SQL for MySQL, PostgreSQL, or SQLite—each with their own syntax variations.

The Builder pattern is especially valuable when the object being constructed has many optional components, when the construction process must allow different representations, or when you want to enforce a specific construction sequence.

## Applicability

Use the Builder pattern when:

- The algorithm for creating a complex object should be independent of the parts that make up the object and how they're assembled
- The construction process must allow different representations for the object that's constructed
- You need to create an object with many optional components or configurations
- You want to avoid "telescoping constructors" (constructors with many parameters)
- You need to construct objects step by step, potentially reusing construction code
- The construction of an object involves many steps that can be combined in different ways
- You want to make the code more readable by using named methods instead of positional constructor arguments

## Structure

```
┌────────────────────────────────────────────────────────────────────────────┐
│                              DIRECTOR                                      │
│  Defines the order of construction steps                                   │
├────────────────────────────────────────────────────────────────────────────┤
│ - builder: Builder                                                         │
├────────────────────────────────────────────────────────────────────────────┤
│ + construct(): void                                                        │
│ + setBuilder(builder: Builder): void                                       │
└────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    │ uses
                                    ▼
┌────────────────────────────────────────────────────────────────────────────┐
│                          «interface»                                       │
│                            BUILDER                                         │
├────────────────────────────────────────────────────────────────────────────┤
│ + buildPartA(): Builder                                                    │
│ + buildPartB(): Builder                                                    │
│ + buildPartC(): Builder                                                    │
│ + getResult(): Product                                                     │
└────────────────────────────────────────────────────────────────────────────┘
                    ▲                               ▲
                    │ implements                    │ implements
        ┌───────────┴───────────────┐   ┌──────────┴────────────────┐
        │                           │   │                           │
┌───────┴───────────────┐   ┌───────┴───────────────┐   ┌───────────┴───────┐
│   ConcreteBuilder1    │   │   ConcreteBuilder2    │   │  ConcreteBuilder3 │
├───────────────────────┤   ├───────────────────────┤   ├───────────────────┤
│ - product: Product1   │   │ - product: Product2   │   │ - product: Product3│
├───────────────────────┤   ├───────────────────────┤   ├───────────────────┤
│ + buildPartA()        │   │ + buildPartA()        │   │ + buildPartA()    │
│ + buildPartB()        │   │ + buildPartB()        │   │ + buildPartB()    │
│ + buildPartC()        │   │ + buildPartC()        │   │ + buildPartC()    │
│ + getResult()         │   │ + getResult()         │   │ + getResult()     │
└───────────────────────┘   └───────────────────────┘   └───────────────────┘
        │                           │                           │
        │ creates                   │ creates                   │ creates
        ▼                           ▼                           ▼
┌───────────────────┐   ┌───────────────────────┐   ┌───────────────────────┐
│     Product1      │   │       Product2        │   │       Product3        │
└───────────────────┘   └───────────────────────┘   └───────────────────────┘


FLUENT BUILDER VARIANT (without Director):

┌────────────────────────────────────────────────────────────────────────────┐
│                            CLIENT                                          │
└────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    │ uses directly
                                    ▼
┌────────────────────────────────────────────────────────────────────────────┐
│                          FluentBuilder                                     │
├────────────────────────────────────────────────────────────────────────────┤
│ - product: Product                                                         │
├────────────────────────────────────────────────────────────────────────────┤
│ + withPartA(value): FluentBuilder    // returns this                       │
│ + withPartB(value): FluentBuilder    // returns this                       │
│ + withPartC(value): FluentBuilder    // returns this                       │
│ + build(): Product                   // returns final product              │
└────────────────────────────────────────────────────────────────────────────┘
```

## Participants

- **Builder** (QueryBuilder interface)
  - Specifies an abstract interface for creating parts of a Product object

- **ConcreteBuilder** (MySqlQueryBuilder, PostgresQueryBuilder)
  - Constructs and assembles parts of the product by implementing the Builder interface
  - Defines and keeps track of the representation it creates
  - Provides an interface to retrieve the product

- **Director** (QueryDirector)
  - Constructs an object using the Builder interface
  - Defines the order in which construction steps are called
  - Optional in the fluent builder variant

- **Product** (Query, SqlStatement)
  - Represents the complex object under construction
  - ConcreteBuilder builds the product's internal representation and defines the process by which it's assembled
  - Includes classes that define the constituent parts

## Collaborations

- The client creates the Director object and configures it with the desired Builder object
- Director notifies the builder whenever a part of the product should be built
- Builder handles requests from the director and adds parts to the product
- The client retrieves the product from the builder

In the fluent variant (without Director):
- The client calls builder methods directly in a chained sequence
- Each method returns the builder, enabling method chaining
- The client calls build() to get the final product

## Consequences

### Benefits

- **Allows variation of product's internal representation**: The Builder object provides the director with an abstract interface for constructing the product. The interface lets the builder hide the representation and internal structure of the product. It also hides how the product gets assembled. Because the product is constructed through an abstract interface, all you have to do to change the product's internal representation is define a new kind of builder.

- **Isolates code for construction and representation**: The Builder pattern improves modularity by encapsulating the way a complex object is constructed and represented. Clients don't need to know anything about the classes that define the product's internal structure; such classes don't appear in Builder's interface.

- **Finer control over construction process**: Unlike creational patterns that construct products in one shot, the Builder pattern constructs the product step by step under the director's control. Only when the product is finished does the director retrieve it from the builder. Hence the Builder interface reflects the process of constructing the product more than other creational patterns.

- **Supports immutable objects**: The Builder pattern works well for creating immutable objects, where all properties must be set during construction.

- **Improved readability**: Method names describe what's being configured, making client code self-documenting.

- **Validation opportunities**: The build() method can validate the configured state before creating the product.

### Liabilities

- **Increased complexity**: The pattern requires creating a separate builder class for each product, which increases the number of classes.

- **Requires mutable builder**: The builder itself must be mutable, even if the product it creates is immutable.

- **Partial products**: If the client doesn't call all necessary methods, the product might be in an incomplete state. This can be mitigated with required parameters in the builder constructor or validation in build().

- **Not suitable for simple objects**: For objects with few properties and no complex construction logic, a simple constructor is clearer.

## Implementation

Consider the following implementation issues:

### 1. Assembly and Construction Interface

Builders construct their products step by step. The Builder class interface must be general enough to allow construction of products for all kinds of concrete builders.

```pseudocode
interface QueryBuilder {
    function select(columns: List<String>): QueryBuilder
    function from(table: String): QueryBuilder
    function where(condition: String): QueryBuilder
    function join(table: String, condition: String): QueryBuilder
    function orderBy(column: String, direction: String): QueryBuilder
    function limit(count: Integer): QueryBuilder
    function offset(start: Integer): QueryBuilder
    function build(): Query
}
```

### 2. Product Representation

The products constructed by different builders can have very different representations. It's often unnecessary to define a common interface for products.

```pseudocode
// Products don't need a common interface
class MySqlQuery {
    private sql: String
    function getSql(): String
    function getParameters(): List<Object>
}

class MongoQuery {
    private collection: String
    private filter: Document
    private projection: Document
    function getCollection(): String
    function getFilter(): Document
}
```

### 3. Why No Abstract Class for Products?

In the typical case, the products produced by the concrete builders differ so greatly in their representation that there is little to gain from giving different products a common parent class.

### 4. Empty Methods as Default

In some cases, methods that build parts of the product should do nothing by default. This allows concrete builders to override only the methods they need.

```pseudocode
class DefaultQueryBuilder implements QueryBuilder {
    protected query: Query = new Query()

    function select(columns: List<String>): QueryBuilder {
        // Default: select all
        return this
    }

    function where(condition: String): QueryBuilder {
        // Default: no where clause
        return this
    }

    function orderBy(column: String, direction: String): QueryBuilder {
        // Default: no ordering
        return this
    }

    // Concrete builders override as needed
}
```

### 5. Fluent Interface Pattern

Modern builders often return `this` from each method, enabling method chaining:

```pseudocode
class HttpRequestBuilder {
    private method: String = "GET"
    private url: String = ""
    private headers: Map<String, String> = {}
    private body: String = null
    private timeout: Integer = 30000

    function withMethod(method: String): HttpRequestBuilder {
        this.method = method
        return this
    }

    function withUrl(url: String): HttpRequestBuilder {
        this.url = url
        return this
    }

    function withHeader(name: String, value: String): HttpRequestBuilder {
        this.headers.put(name, value)
        return this
    }

    function withBody(body: String): HttpRequestBuilder {
        this.body = body
        return this
    }

    function withTimeout(timeout: Integer): HttpRequestBuilder {
        this.timeout = timeout
        return this
    }

    function build(): HttpRequest {
        validate()
        return new HttpRequest(method, url, headers, body, timeout)
    }

    private function validate() {
        if (url.isEmpty()) {
            throw new InvalidRequestException("URL is required")
        }
        if (method in ["POST", "PUT", "PATCH"] && body == null) {
            throw new InvalidRequestException("Body required for " + method)
        }
    }
}

// Usage
request = new HttpRequestBuilder()
    .withMethod("POST")
    .withUrl("https://api.example.com/users")
    .withHeader("Content-Type", "application/json")
    .withHeader("Authorization", "Bearer token123")
    .withBody('{"name": "John", "email": "john@example.com"}')
    .withTimeout(5000)
    .build()
```

### 6. Step Builder for Enforced Order

When certain build steps are required or must happen in order, use a step builder:

```pseudocode
// Interfaces enforce the build order
interface NeedsFrom {
    function from(table: String): NeedsSelect
}

interface NeedsSelect {
    function select(columns: List<String>): OptionalClauses
}

interface OptionalClauses {
    function where(condition: String): OptionalClauses
    function orderBy(column: String): OptionalClauses
    function build(): Query
}

class StepQueryBuilder implements NeedsFrom, NeedsSelect, OptionalClauses {
    private tableName: String
    private columns: List<String>
    private conditions: List<String> = []
    private ordering: List<String> = []

    // Step 1: Must specify table first
    function from(table: String): NeedsSelect {
        this.tableName = table
        return this
    }

    // Step 2: Must specify columns
    function select(columns: List<String>): OptionalClauses {
        this.columns = columns
        return this
    }

    // Step 3+: Optional clauses
    function where(condition: String): OptionalClauses {
        this.conditions.add(condition)
        return this
    }

    function orderBy(column: String): OptionalClauses {
        this.ordering.add(column)
        return this
    }

    function build(): Query {
        return new Query(tableName, columns, conditions, ordering)
    }
}

// Usage - compiler enforces the order
query = new StepQueryBuilder()
    .from("users")           // Must come first
    .select(["id", "name"])  // Must come second
    .where("active = true")  // Optional
    .orderBy("name")         // Optional
    .build()

// This won't compile:
// new StepQueryBuilder().select(["id"])  // Error: from() must come first
```

### 7. Builder with Director

```pseudocode
class QueryDirector {
    private builder: QueryBuilder

    function setBuilder(builder: QueryBuilder) {
        this.builder = builder
    }

    function buildMinimalQuery(table: String): Query {
        return builder
            .from(table)
            .select(["*"])
            .build()
    }

    function buildPaginatedQuery(table: String, page: Integer, pageSize: Integer): Query {
        offset = (page - 1) * pageSize
        return builder
            .from(table)
            .select(["*"])
            .limit(pageSize)
            .offset(offset)
            .build()
    }

    function buildSearchQuery(table: String, searchColumn: String, searchTerm: String): Query {
        return builder
            .from(table)
            .select(["*"])
            .where(searchColumn + " LIKE '%" + searchTerm + "%'")
            .orderBy(searchColumn, "ASC")
            .build()
    }
}
```

## Example

Here's a complete example of the Builder pattern applied to constructing email messages with multiple optional components:

```pseudocode
// ============================================================
// PRODUCT
// ============================================================

class Email {
    private from: String
    private to: List<String>
    private cc: List<String>
    private bcc: List<String>
    private subject: String
    private textBody: String
    private htmlBody: String
    private attachments: List<Attachment>
    private headers: Map<String, String>
    private priority: EmailPriority
    private replyTo: String
    private readReceipt: Boolean

    // Package-private constructor - only Builder can create
    constructor(builder: EmailBuilder) {
        this.from = builder.from
        this.to = List.copyOf(builder.to)
        this.cc = List.copyOf(builder.cc)
        this.bcc = List.copyOf(builder.bcc)
        this.subject = builder.subject
        this.textBody = builder.textBody
        this.htmlBody = builder.htmlBody
        this.attachments = List.copyOf(builder.attachments)
        this.headers = Map.copyOf(builder.headers)
        this.priority = builder.priority
        this.replyTo = builder.replyTo
        this.readReceipt = builder.readReceipt
    }

    // Getters only - Email is immutable
    function getFrom(): String { return from }
    function getTo(): List<String> { return to }
    function getCc(): List<String> { return cc }
    function getBcc(): List<String> { return bcc }
    function getSubject(): String { return subject }
    function getTextBody(): String { return textBody }
    function getHtmlBody(): String { return htmlBody }
    function getAttachments(): List<Attachment> { return attachments }
    function getHeaders(): Map<String, String> { return headers }
    function getPriority(): EmailPriority { return priority }
    function getReplyTo(): String { return replyTo }
    function requestsReadReceipt(): Boolean { return readReceipt }

    function hasHtmlBody(): Boolean {
        return htmlBody != null && !htmlBody.isEmpty()
    }

    function isMultipart(): Boolean {
        return hasHtmlBody() || attachments.size() > 0
    }
}

class Attachment {
    private filename: String
    private contentType: String
    private data: Bytes
    private isInline: Boolean
    private contentId: String

    constructor(filename: String, contentType: String, data: Bytes) {
        this.filename = filename
        this.contentType = contentType
        this.data = data
        this.isInline = false
        this.contentId = null
    }

    function asInline(contentId: String): Attachment {
        this.isInline = true
        this.contentId = contentId
        return this
    }

    // Getters...
}

enum EmailPriority {
    LOW,
    NORMAL,
    HIGH,
    URGENT
}

// ============================================================
// BUILDER
// ============================================================

class EmailBuilder {
    // Package-visible for Email constructor
    from: String = null
    to: List<String> = []
    cc: List<String> = []
    bcc: List<String> = []
    subject: String = ""
    textBody: String = null
    htmlBody: String = null
    attachments: List<Attachment> = []
    headers: Map<String, String> = {}
    priority: EmailPriority = EmailPriority.NORMAL
    replyTo: String = null
    readReceipt: Boolean = false

    // Required field - passed to constructor
    constructor(from: String) {
        if (from == null || !isValidEmail(from)) {
            throw new InvalidEmailException("Valid 'from' address required")
        }
        this.from = from
    }

    // Static factory method alternative
    static function from(address: String): EmailBuilder {
        return new EmailBuilder(address)
    }

    function to(address: String): EmailBuilder {
        validateEmail(address)
        this.to.add(address)
        return this
    }

    function to(addresses: List<String>): EmailBuilder {
        for (address in addresses) {
            validateEmail(address)
            this.to.add(address)
        }
        return this
    }

    function cc(address: String): EmailBuilder {
        validateEmail(address)
        this.cc.add(address)
        return this
    }

    function cc(addresses: List<String>): EmailBuilder {
        for (address in addresses) {
            validateEmail(address)
            this.cc.add(address)
        }
        return this
    }

    function bcc(address: String): EmailBuilder {
        validateEmail(address)
        this.bcc.add(address)
        return this
    }

    function subject(subject: String): EmailBuilder {
        this.subject = subject ?? ""
        return this
    }

    function textBody(body: String): EmailBuilder {
        this.textBody = body
        return this
    }

    function htmlBody(body: String): EmailBuilder {
        this.htmlBody = body
        return this
    }

    function body(text: String, html: String): EmailBuilder {
        this.textBody = text
        this.htmlBody = html
        return this
    }

    function attach(filename: String, contentType: String, data: Bytes): EmailBuilder {
        attachment = new Attachment(filename, contentType, data)
        this.attachments.add(attachment)
        return this
    }

    function attachFile(file: File): EmailBuilder {
        contentType = MimeTypes.guessFromFilename(file.getName())
        data = file.readBytes()
        return attach(file.getName(), contentType, data)
    }

    function attachInline(contentId: String, filename: String,
                          contentType: String, data: Bytes): EmailBuilder {
        attachment = new Attachment(filename, contentType, data)
        attachment.asInline(contentId)
        this.attachments.add(attachment)
        return this
    }

    function header(name: String, value: String): EmailBuilder {
        this.headers.put(name, value)
        return this
    }

    function priority(priority: EmailPriority): EmailBuilder {
        this.priority = priority
        return this
    }

    function highPriority(): EmailBuilder {
        return priority(EmailPriority.HIGH)
    }

    function lowPriority(): EmailBuilder {
        return priority(EmailPriority.LOW)
    }

    function replyTo(address: String): EmailBuilder {
        validateEmail(address)
        this.replyTo = address
        return this
    }

    function requestReadReceipt(): EmailBuilder {
        this.readReceipt = true
        return this
    }

    function build(): Email {
        validate()
        return new Email(this)
    }

    private function validate() {
        errors = []

        if (to.isEmpty()) {
            errors.add("At least one recipient (to) is required")
        }

        if (textBody == null && htmlBody == null) {
            errors.add("Email must have a text body or HTML body")
        }

        if (subject.isEmpty()) {
            errors.add("Subject is required")
        }

        totalAttachmentSize = attachments
            .map(a => a.getData().length)
            .sum()
        if (totalAttachmentSize > 25 * 1024 * 1024) {  // 25MB
            errors.add("Total attachment size exceeds 25MB limit")
        }

        if (!errors.isEmpty()) {
            throw new InvalidEmailException(errors.join("; "))
        }
    }

    private function validateEmail(address: String) {
        if (address == null || !isValidEmail(address)) {
            throw new InvalidEmailException("Invalid email: " + address)
        }
    }

    private function isValidEmail(address: String): Boolean {
        // Basic email validation
        return address.matches("^[^@]+@[^@]+\\.[^@]+$")
    }
}

// ============================================================
// DIRECTOR (Optional - for common email types)
// ============================================================

class EmailDirector {
    private smtpConfig: SmtpConfiguration

    constructor(config: SmtpConfiguration) {
        this.smtpConfig = config
    }

    function createWelcomeEmail(toAddress: String, userName: String): Email {
        template = TemplateEngine.load("welcome-email")

        return EmailBuilder.from(smtpConfig.getDefaultFrom())
            .to(toAddress)
            .subject("Welcome to Our Platform, " + userName + "!")
            .textBody(template.renderText({"name": userName}))
            .htmlBody(template.renderHtml({"name": userName}))
            .header("X-Email-Type", "welcome")
            .build()
    }

    function createPasswordResetEmail(toAddress: String, resetLink: String): Email {
        template = TemplateEngine.load("password-reset")

        return EmailBuilder.from(smtpConfig.getSecurityFrom())
            .to(toAddress)
            .subject("Password Reset Request")
            .textBody(template.renderText({"link": resetLink}))
            .htmlBody(template.renderHtml({"link": resetLink}))
            .highPriority()
            .header("X-Email-Type", "security")
            .build()
    }

    function createOrderConfirmation(toAddress: String, order: Order): Email {
        template = TemplateEngine.load("order-confirmation")
        data = {
            "orderNumber": order.getNumber(),
            "items": order.getItems(),
            "total": order.getTotal()
        }

        builder = EmailBuilder.from(smtpConfig.getOrdersFrom())
            .to(toAddress)
            .subject("Order Confirmation #" + order.getNumber())
            .textBody(template.renderText(data))
            .htmlBody(template.renderHtml(data))
            .header("X-Order-Number", order.getNumber())

        // Attach invoice PDF
        invoice = InvoiceGenerator.generate(order)
        builder.attach(
            "invoice-" + order.getNumber() + ".pdf",
            "application/pdf",
            invoice.toBytes()
        )

        return builder.build()
    }

    function createNewsletterEmail(recipients: List<String>,
                                    subject: String,
                                    content: NewsletterContent): Email {
        builder = EmailBuilder.from(smtpConfig.getNewsletterFrom())
            .subject(subject)
            .textBody(content.getTextVersion())
            .htmlBody(content.getHtmlVersion())
            .header("List-Unsubscribe", content.getUnsubscribeUrl())
            .header("X-Email-Type", "newsletter")
            .lowPriority()

        for (recipient in recipients) {
            builder.bcc(recipient)
        }

        return builder.build()
    }
}

// ============================================================
// CONCRETE BUILDERS FOR DIFFERENT FORMATS
// ============================================================

interface EmailFormatBuilder {
    function setFrom(from: String): EmailFormatBuilder
    function setTo(to: List<String>): EmailFormatBuilder
    function setCc(cc: List<String>): EmailFormatBuilder
    function setSubject(subject: String): EmailFormatBuilder
    function setTextBody(body: String): EmailFormatBuilder
    function setHtmlBody(body: String): EmailFormatBuilder
    function setHeaders(headers: Map<String, String>): EmailFormatBuilder
    function build(): String
}

class MimeEmailBuilder implements EmailFormatBuilder {
    private parts: List<String> = []
    private boundary: String = generateBoundary()

    function setFrom(from: String): EmailFormatBuilder {
        parts.add("From: " + from)
        return this
    }

    function setTo(to: List<String>): EmailFormatBuilder {
        parts.add("To: " + to.join(", "))
        return this
    }

    function setCc(cc: List<String>): EmailFormatBuilder {
        if (!cc.isEmpty()) {
            parts.add("Cc: " + cc.join(", "))
        }
        return this
    }

    function setSubject(subject: String): EmailFormatBuilder {
        // Encode if needed
        encoded = MimeEncoder.encodeIfNecessary(subject)
        parts.add("Subject: " + encoded)
        return this
    }

    function setTextBody(body: String): EmailFormatBuilder {
        if (body != null) {
            parts.add("--" + boundary)
            parts.add("Content-Type: text/plain; charset=UTF-8")
            parts.add("Content-Transfer-Encoding: quoted-printable")
            parts.add("")
            parts.add(QuotedPrintable.encode(body))
        }
        return this
    }

    function setHtmlBody(body: String): EmailFormatBuilder {
        if (body != null) {
            parts.add("--" + boundary)
            parts.add("Content-Type: text/html; charset=UTF-8")
            parts.add("Content-Transfer-Encoding: quoted-printable")
            parts.add("")
            parts.add(QuotedPrintable.encode(body))
        }
        return this
    }

    function setHeaders(headers: Map<String, String>): EmailFormatBuilder {
        for ((name, value) in headers) {
            parts.add(name + ": " + value)
        }
        return this
    }

    function build(): String {
        parts.add("--" + boundary + "--")
        return parts.join("\r\n")
    }

    private function generateBoundary(): String {
        return "----=_Part_" + UUID.randomUUID().toString()
    }
}

class SendGridApiBuilder implements EmailFormatBuilder {
    private payload: Map<String, Object> = {}

    function setFrom(from: String): EmailFormatBuilder {
        payload["from"] = {"email": from}
        return this
    }

    function setTo(to: List<String>): EmailFormatBuilder {
        personalizations = payload.getOrDefault("personalizations", [{}])
        personalizations[0]["to"] = to.map(email => {"email": email})
        payload["personalizations"] = personalizations
        return this
    }

    function setCc(cc: List<String>): EmailFormatBuilder {
        if (!cc.isEmpty()) {
            personalizations = payload.getOrDefault("personalizations", [{}])
            personalizations[0]["cc"] = cc.map(email => {"email": email})
            payload["personalizations"] = personalizations
        }
        return this
    }

    function setSubject(subject: String): EmailFormatBuilder {
        payload["subject"] = subject
        return this
    }

    function setTextBody(body: String): EmailFormatBuilder {
        if (body != null) {
            content = payload.getOrDefault("content", [])
            content.add({"type": "text/plain", "value": body})
            payload["content"] = content
        }
        return this
    }

    function setHtmlBody(body: String): EmailFormatBuilder {
        if (body != null) {
            content = payload.getOrDefault("content", [])
            content.add({"type": "text/html", "value": body})
            payload["content"] = content
        }
        return this
    }

    function setHeaders(headers: Map<String, String>): EmailFormatBuilder {
        if (!headers.isEmpty()) {
            payload["headers"] = headers
        }
        return this
    }

    function build(): String {
        return JSON.stringify(payload)
    }
}

// ============================================================
// USAGE EXAMPLES
// ============================================================

function main() {
    // Example 1: Simple email with fluent builder
    email = EmailBuilder.from("sender@example.com")
        .to("recipient@example.com")
        .subject("Meeting Tomorrow")
        .textBody("Hi,\n\nJust a reminder about our meeting tomorrow at 2pm.\n\nBest regards")
        .build()

    // Example 2: Complex email with multiple features
    complexEmail = EmailBuilder.from("noreply@company.com")
        .to("customer@example.com")
        .cc("sales@company.com")
        .bcc("records@company.com")
        .subject("Your Invoice #12345")
        .textBody("Please find your invoice attached.")
        .htmlBody("<html><body><h1>Invoice</h1><p>Please find your invoice attached.</p></body></html>")
        .attachFile(new File("/invoices/12345.pdf"))
        .highPriority()
        .replyTo("billing@company.com")
        .requestReadReceipt()
        .header("X-Invoice-Number", "12345")
        .build()

    // Example 3: Using Director for common patterns
    config = SmtpConfiguration.load()
    director = new EmailDirector(config)

    welcomeEmail = director.createWelcomeEmail("newuser@example.com", "John")
    resetEmail = director.createPasswordResetEmail("user@example.com", "https://reset.link/abc123")

    // Example 4: Building different formats from same Email
    mimeBuilder = new MimeEmailBuilder()
    mimeFormat = buildEmailFormat(email, mimeBuilder)

    sendGridBuilder = new SendGridApiBuilder()
    sendGridFormat = buildEmailFormat(email, sendGridBuilder)
}

function buildEmailFormat(email: Email, builder: EmailFormatBuilder): String {
    return builder
        .setFrom(email.getFrom())
        .setTo(email.getTo())
        .setCc(email.getCc())
        .setSubject(email.getSubject())
        .setTextBody(email.getTextBody())
        .setHtmlBody(email.getHtmlBody())
        .setHeaders(email.getHeaders())
        .build()
}
```

## Known Uses

- **StringBuilder/StringBuffer**: In Java and many languages, StringBuilder uses a builder-like pattern to efficiently construct strings through append operations.

- **SQL Query Builders**: Nearly every ORM and database library uses Builder (Doctrine QueryBuilder, Eloquent, Knex.js, jOOQ, SQLAlchemy).

- **HTTP Client Libraries**: OkHttp, Apache HttpClient, and others use builders for constructing requests (Request.Builder, HttpRequest.Builder).

- **Protocol Buffers**: Google's protobuf generates builder classes for each message type.

- **GUI Frameworks**: Swing's JFrame/JPanel setup, JavaFX, Flutter's widget tree, and SwiftUI all use builder-like patterns.

- **Lombok @Builder**: The Lombok library generates builder classes from Java annotations.

- **Calendar/DateTime APIs**: Java's Calendar.Builder, Java 8+ DateTimeFormatterBuilder.

- **JSON Libraries**: Gson's GsonBuilder, Jackson's ObjectMapper configuration.

- **Test Fixtures**: Many testing frameworks use builders for creating test data (TestDataBuilder pattern).

- **Configuration Objects**: Most framework configurations use builders (Spring Security, Log4j2, Hibernate).

## Related Patterns

- **Abstract Factory**: Abstract Factory is similar to Builder in that it too may construct complex objects. The primary difference is that the Builder pattern focuses on constructing a complex object step by step, while Abstract Factory emphasizes families of product objects. Builder returns the product as a final step, but Abstract Factory returns the product immediately.

- **Composite**: Builders are often used to create Composite structures, building the tree step by step.

- **Prototype**: Builder can use Prototype for creating parts of complex objects, cloning prototypes rather than constructing from scratch.

- **Fluent Interface**: Modern builders typically implement the fluent interface pattern, returning `this` from each method to enable method chaining.

- **Factory Method**: Director might use Factory Methods to determine which ConcreteBuilder to instantiate.

- **Singleton**: Directors are often Singletons, and builders might be configured as Singletons when they're stateless.

## When NOT to Use

- **Simple objects**: If an object can be created with a straightforward constructor call with a few parameters, a builder adds unnecessary complexity.

- **Objects with required fields only**: If all fields are required with no optional ones, a constructor with named parameters (in languages that support them) is clearer.

- **Immutability not required**: If the object can be modified after creation, you might use setters directly on the object instead of a builder.

- **Performance-critical loops**: Creating a builder object adds allocation overhead. In tight loops, direct construction may be faster.

- **Small configuration surface**: If there are only 2-3 configuration options, a constructor with default parameters or a factory method is simpler.

**Simpler alternatives**:

- **Constructor with named/optional parameters**: Languages like Python, Kotlin, or C# allow named parameters with defaults, reducing the need for builders
  ```
  email = Email(from="a@b.com", to="c@d.com", subject="Hi")
  ```

- **Factory Methods**: For creating objects with a few common configurations
  ```
  Email.createSimple(from, to, subject, body)
  Email.createWithAttachments(from, to, subject, body, attachments)
  ```

- **Telescoping constructors**: For objects with few variations (though this becomes unwieldy quickly)

- **Object initializer syntax**: C# and similar languages allow setting properties during construction
  ```
  new Email { From = "a@b.com", To = "c@d.com" }
  ```

**Signs you've over-engineered**:
- Your builder has only 2-3 methods
- You never use optional configuration - always call every method
- The product class is simple with no complex construction logic
- You're building a builder for a data transfer object with only fields
- The builder is used in only one place in your codebase

---

*Based on concepts from "Design Patterns: Elements of Reusable Object-Oriented Software" by Gamma, Helm, Johnson, and Vlissides (Gang of Four), 1994.*
