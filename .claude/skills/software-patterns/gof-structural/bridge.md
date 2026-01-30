# Bridge

## Intent

Decouple an abstraction from its implementation so that the two can vary independently. Bridge separates a large class or a set of closely related classes into two separate hierarchies - abstraction and implementation - which can be developed independently of each other.

## Also Known As

- Handle/Body
- Pimpl (Pointer to Implementation)

## Motivation

Consider a drawing application that needs to support multiple shapes (circles, rectangles, triangles) across multiple rendering platforms (Windows, macOS, Linux, or different rendering engines like OpenGL, DirectX, SVG). Without the Bridge pattern, you might create a class hierarchy where each combination requires its own class: `WindowsCircle`, `MacOSCircle`, `LinuxCircle`, `WindowsRectangle`, `MacOSRectangle`, and so on.

This approach leads to a combinatorial explosion of classes. If you have 5 shapes and 4 platforms, you need 20 classes. Adding a new shape requires 4 new classes, and adding a new platform requires 5 new classes. The inheritance hierarchy becomes unwieldy and difficult to maintain.

The Bridge pattern addresses this by separating the shape abstraction from the rendering implementation. The shape hierarchy defines what to draw (circle, rectangle), while the renderer hierarchy defines how to draw it (OpenGL, SVG, Canvas). Each shape holds a reference to a renderer and delegates the platform-specific work to it. Now you have 5 shape classes and 4 renderer classes (9 total) instead of 20, and adding a new shape or renderer requires only one new class.

This separation also enables you to change the rendering implementation at runtime. The same circle object could be rendered using OpenGL for performance or SVG for scalability, simply by swapping the renderer reference.

## Applicability

Use the Bridge pattern when:

- You want to avoid a permanent binding between an abstraction and its implementation. This might be the case when the implementation must be selected or switched at runtime.

- Both the abstractions and their implementations should be extensible by subclassing. The Bridge pattern lets you combine different abstractions and implementations and extend them independently.

- Changes in the implementation of an abstraction should have no impact on clients; that is, their code should not have to be recompiled.

- You have a proliferation of classes resulting from a coupled abstraction and implementation. The Bridge pattern lets you split the monolithic class hierarchy into two separate hierarchies.

- You want to share an implementation among multiple objects (perhaps using reference counting), and this fact should be hidden from the client.

- You need to map orthogonal class hierarchies. When you notice that a class needs to be extended in two independent dimensions, Bridge lets you manage each dimension separately.

## Structure

```
┌─────────────────────────┐           ┌─────────────────────────┐
│      Abstraction        │           │    <<interface>>        │
├─────────────────────────┤           │     Implementor         │
│ - impl: Implementor     │──────────▶├─────────────────────────┤
├─────────────────────────┤           │ + operationImpl()       │
│ + operation()           │           └─────────────────────────┘
│   impl.operationImpl()  │                       △
└─────────────────────────┘                       │
            △                          ┌──────────┴──────────┐
            │                          │                     │
┌───────────┴───────────┐    ┌─────────────────┐   ┌─────────────────┐
│  RefinedAbstraction   │    │ConcreteImplA    │   │ConcreteImplB    │
├───────────────────────┤    ├─────────────────┤   ├─────────────────┤
│ + refinedOperation()  │    │+ operationImpl()│   │+ operationImpl()│
└───────────────────────┘    └─────────────────┘   └─────────────────┘

       Abstraction                    Implementation
       Hierarchy                      Hierarchy
```

### Detailed Structure

```
                         Client
                            │
                            ▼
┌────────────────────────────────────────────────────────────────────┐
│                        Abstraction                                  │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  RemoteControl                                                │  │
│  │  ─────────────                                                │  │
│  │  - device: Device                                             │  │
│  │  ─────────────                                                │  │
│  │  + togglePower()                                              │  │
│  │  + volumeUp()                                                 │  │
│  │  + volumeDown()                                               │  │
│  │  + channelUp()                                                │  │
│  │  + channelDown()                                              │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                              △                                      │
│                              │                                      │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  AdvancedRemoteControl                                        │  │
│  │  ─────────────────────                                        │  │
│  │  + mute()                                                     │  │
│  │  + setChannel(channel)                                        │  │
│  └──────────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────────┘
                            │
                            │ uses
                            ▼
┌────────────────────────────────────────────────────────────────────┐
│                       Implementation                                │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  <<interface>> Device                                         │  │
│  │  ────────────────────                                         │  │
│  │  + isEnabled(): Boolean                                       │  │
│  │  + enable()                                                   │  │
│  │  + disable()                                                  │  │
│  │  + getVolume(): Integer                                       │  │
│  │  + setVolume(volume)                                          │  │
│  │  + getChannel(): Integer                                      │  │
│  │  + setChannel(channel)                                        │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                              △                                      │
│              ┌───────────────┼───────────────┐                     │
│              │               │               │                     │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐          │
│  │      TV       │  │    Radio      │  │  Soundbar     │          │
│  └───────────────┘  └───────────────┘  └───────────────┘          │
└────────────────────────────────────────────────────────────────────┘
```

## Participants

- **Abstraction**: Defines the abstraction's interface. Maintains a reference to an object of type Implementor. The abstraction delegates work to the implementation object.

- **RefinedAbstraction**: Extends the interface defined by Abstraction. Adds additional operations or specializes existing ones while still delegating implementation work.

- **Implementor**: Defines the interface for implementation classes. This interface does not have to correspond exactly to the Abstraction's interface; in fact, the two interfaces can be quite different. Typically the Implementor interface provides only primitive operations, and Abstraction defines higher-level operations based on these primitives.

- **ConcreteImplementor**: Implements the Implementor interface and defines its concrete implementation. Each ConcreteImplementor corresponds to a specific platform or implementation strategy.

## Collaborations

1. The client interacts with the Abstraction through its public interface.

2. The Abstraction maintains a reference to an Implementor object and forwards client requests to this object.

3. The Abstraction may perform additional work before or after delegating to the Implementor, combining primitive operations into higher-level ones.

4. ConcreteImplementors handle the actual platform-specific or algorithm-specific work.

5. The Abstraction and Implementor hierarchies can be extended independently without affecting each other.

## Consequences

### Benefits

- **Decoupling interface and implementation**: The implementation can be selected and changed at runtime. Abstraction and implementation can evolve independently.

- **Improved extensibility**: You can extend the Abstraction and Implementor hierarchies independently. Adding a new refined abstraction does not require changes to implementors, and vice versa.

- **Hiding implementation details**: Clients are shielded from implementation details. Changes to the implementation do not require recompiling the abstraction or client code.

- **Reduced class explosion**: Instead of creating a class for every combination of abstraction and implementation, you create separate hierarchies. This reduces the total number of classes from M * N to M + N.

- **Sharing implementations**: Multiple abstractions can share the same implementation object, enabling resource sharing and consistency.

- **Runtime flexibility**: You can switch implementations at runtime, enabling dynamic behavior changes based on context, configuration, or user preferences.

- **Cleaner code organization**: Separates high-level logic from low-level details, making both easier to understand and maintain.

### Liabilities

- **Increased complexity**: The pattern introduces additional indirection, which can make code harder to follow for simple cases where a single hierarchy would suffice.

- **Double dispatch overhead**: Each operation involves at least two method calls (abstraction to implementor), which may have a minor performance impact.

- **Design difficulty**: Identifying the right split between abstraction and implementation requires careful analysis. A wrong split can lead to awkward code.

- **Risk of over-engineering**: For systems with limited variation, Bridge adds unnecessary complexity. If you only have one implementation, the pattern provides little benefit.

- **Coordination challenges**: When both hierarchies evolve, ensuring they remain compatible requires careful coordination and testing.

## Implementation

### Implementation Considerations

1. **Only one Implementor**: If there is only one implementation, creating an abstract Implementor class may be unnecessary. This is a degenerate case of Bridge that still provides the benefit of hiding implementation from clients.

2. **Creating the right Implementor**: How and when does the Abstraction get its Implementor? Common approaches include:
   - Constructor parameter (dependency injection)
   - Factory method that chooses based on parameters
   - Default implementation that can be changed later
   - Configuration file or environment-based selection

3. **Sharing Implementors**: Multiple Abstraction objects can share the same Implementor using reference counting or a flyweight pattern.

4. **Using multiple Implementors**: An Abstraction could use multiple Implementors for different aspects of its functionality.

### Basic Bridge Implementation

```
// Implementor interface - the "how"
interface Renderer
    method renderCircle(x: Float, y: Float, radius: Float)
    method renderRectangle(x: Float, y: Float, width: Float, height: Float)
    method renderLine(x1: Float, y1: Float, x2: Float, y2: Float)
    method setColor(r: Integer, g: Integer, b: Integer)
end interface

// Concrete Implementor A
class OpenGLRenderer implements Renderer
    method renderCircle(x: Float, y: Float, radius: Float)
        print "OpenGL: Drawing circle at (" + x + "," + y + ") with radius " + radius
        // OpenGL-specific circle rendering code
        // glBegin(GL_TRIANGLE_FAN), etc.
    end method

    method renderRectangle(x: Float, y: Float, width: Float, height: Float)
        print "OpenGL: Drawing rectangle at (" + x + "," + y + ")"
        // OpenGL-specific rectangle rendering
    end method

    method renderLine(x1: Float, y1: Float, x2: Float, y2: Float)
        print "OpenGL: Drawing line from (" + x1 + "," + y1 + ") to (" + x2 + "," + y2 + ")"
    end method

    method setColor(r: Integer, g: Integer, b: Integer)
        // glColor3f(r/255.0, g/255.0, b/255.0)
    end method
end class

// Concrete Implementor B
class SVGRenderer implements Renderer
    private svgElements: List

    constructor()
        this.svgElements = new List()
    end constructor

    method renderCircle(x: Float, y: Float, radius: Float)
        svg = "<circle cx=\"" + x + "\" cy=\"" + y + "\" r=\"" + radius + "\"/>"
        svgElements.add(svg)
        print "SVG: " + svg
    end method

    method renderRectangle(x: Float, y: Float, width: Float, height: Float)
        svg = "<rect x=\"" + x + "\" y=\"" + y + "\" width=\"" + width + "\" height=\"" + height + "\"/>"
        svgElements.add(svg)
        print "SVG: " + svg
    end method

    method renderLine(x1: Float, y1: Float, x2: Float, y2: Float)
        svg = "<line x1=\"" + x1 + "\" y1=\"" + y1 + "\" x2=\"" + x2 + "\" y2=\"" + y2 + "\"/>"
        svgElements.add(svg)
    end method

    method setColor(r: Integer, g: Integer, b: Integer)
        // Store color for next element's fill/stroke attributes
    end method

    method exportSVG(): String
        return "<svg>" + svgElements.join("") + "</svg>"
    end method
end class

// Concrete Implementor C
class CanvasRenderer implements Renderer
    private context: CanvasContext

    constructor(canvas: HTMLCanvas)
        this.context = canvas.getContext("2d")
    end constructor

    method renderCircle(x: Float, y: Float, radius: Float)
        context.beginPath()
        context.arc(x, y, radius, 0, 2 * PI)
        context.fill()
        print "Canvas: Drawing circle at (" + x + "," + y + ")"
    end method

    method renderRectangle(x: Float, y: Float, width: Float, height: Float)
        context.fillRect(x, y, width, height)
        print "Canvas: Drawing rectangle"
    end method

    method renderLine(x1: Float, y1: Float, x2: Float, y2: Float)
        context.beginPath()
        context.moveTo(x1, y1)
        context.lineTo(x2, y2)
        context.stroke()
    end method

    method setColor(r: Integer, g: Integer, b: Integer)
        context.fillStyle = "rgb(" + r + "," + g + "," + b + ")"
    end method
end class

// Abstraction - the "what"
abstract class Shape
    protected renderer: Renderer
    protected x: Float
    protected y: Float
    protected color: Color

    constructor(renderer: Renderer)
        this.renderer = renderer
    end constructor

    method setPosition(x: Float, y: Float)
        this.x = x
        this.y = y
    end method

    method setColor(color: Color)
        this.color = color
    end method

    method setRenderer(renderer: Renderer)
        this.renderer = renderer
    end method

    abstract method draw()
    abstract method resize(factor: Float)
end class

// Refined Abstraction A
class Circle extends Shape
    private radius: Float

    constructor(renderer: Renderer, radius: Float)
        super(renderer)
        this.radius = radius
    end constructor

    method draw()
        if color != null
            renderer.setColor(color.r, color.g, color.b)
        end if
        renderer.renderCircle(x, y, radius)
    end method

    method resize(factor: Float)
        radius = radius * factor
    end method

    method getArea(): Float
        return PI * radius * radius
    end method
end class

// Refined Abstraction B
class Rectangle extends Shape
    private width: Float
    private height: Float

    constructor(renderer: Renderer, width: Float, height: Float)
        super(renderer)
        this.width = width
        this.height = height
    end constructor

    method draw()
        if color != null
            renderer.setColor(color.r, color.g, color.b)
        end if
        renderer.renderRectangle(x, y, width, height)
    end method

    method resize(factor: Float)
        width = width * factor
        height = height * factor
    end method

    method getArea(): Float
        return width * height
    end method
end class

// Usage
openglRenderer = new OpenGLRenderer()
svgRenderer = new SVGRenderer()

// Create shapes with OpenGL renderer
circle = new Circle(openglRenderer, 5.0)
circle.setPosition(100, 100)
circle.draw()  // Uses OpenGL

// Switch renderer at runtime
circle.setRenderer(svgRenderer)
circle.draw()  // Now uses SVG

// Create rectangle with SVG renderer
rectangle = new Rectangle(svgRenderer, 50, 30)
rectangle.setPosition(200, 200)
rectangle.draw()
```

### Bridge with Dependency Injection

```
// Configuration-driven implementor selection
class RendererFactory
    method createRenderer(config: Configuration): Renderer
        renderingEngine = config.get("rendering.engine")

        switch renderingEngine
            case "opengl":
                return new OpenGLRenderer()
            case "svg":
                return new SVGRenderer()
            case "canvas":
                canvas = document.getElementById(config.get("canvas.id"))
                return new CanvasRenderer(canvas)
            case "pdf":
                return new PDFRenderer(config.get("pdf.output.path"))
            default:
                throw new UnknownRendererException(renderingEngine)
        end switch
    end method
end class

// Dependency injection container
class DrawingApplication
    private renderer: Renderer
    private shapes: List<Shape>

    constructor(renderer: Renderer)
        this.renderer = renderer
        this.shapes = new List()
    end constructor

    method addShape(shapeType: String, params: Object): Shape
        shape = ShapeFactory.create(shapeType, renderer, params)
        shapes.add(shape)
        return shape
    end method

    method render()
        for each shape in shapes
            shape.draw()
        end for
    end method

    // Switch all shapes to a new renderer
    method switchRenderer(newRenderer: Renderer)
        this.renderer = newRenderer
        for each shape in shapes
            shape.setRenderer(newRenderer)
        end for
    end method
end class
```

## Example

### Cross-Platform Notification System

A real-world example where a notification system must work across multiple platforms (mobile push, email, SMS) with different notification types (alerts, reminders, marketing).

```
// Implementor - notification delivery mechanism
interface NotificationSender
    method send(
        recipient: String,
        title: String,
        body: String,
        metadata: Map
    ): SendResult

    method sendBatch(notifications: List<NotificationData>): BatchResult

    method getDeliveryStatus(messageId: String): DeliveryStatus

    method getCapabilities(): SenderCapabilities
end interface

class SenderCapabilities
    public supportsRichContent: Boolean
    public supportsAttachments: Boolean
    public maxBodyLength: Integer
    public supportsScheduling: Boolean
    public supportsBatching: Boolean
end class

class SendResult
    public success: Boolean
    public messageId: String
    public error: String
    public timestamp: DateTime
end class

// Concrete Implementor: Push Notifications
class PushNotificationSender implements NotificationSender
    private firebaseClient: FirebaseClient
    private apnsClient: APNSClient

    constructor(firebaseCredentials: Object, apnsCredentials: Object)
        this.firebaseClient = new FirebaseClient(firebaseCredentials)
        this.apnsClient = new APNSClient(apnsCredentials)
    end constructor

    method send(
        recipient: String,
        title: String,
        body: String,
        metadata: Map
    ): SendResult
        // Determine platform from recipient token format
        if recipient.startsWith("fcm:")
            token = recipient.substring(4)
            return sendFirebase(token, title, body, metadata)
        else if recipient.startsWith("apns:")
            token = recipient.substring(5)
            return sendAPNS(token, title, body, metadata)
        end if

        return SendResult.failure("Unknown push token format")
    end method

    private method sendFirebase(token: String, title: String, body: String, metadata: Map): SendResult
        message = {
            token: token,
            notification: { title: title, body: body },
            data: metadata,
            android: {
                priority: metadata.get("priority") or "high",
                notification: { sound: "default" }
            }
        }

        try
            response = firebaseClient.send(message)
            return SendResult.success(response.messageId)
        catch error
            return SendResult.failure(error.message)
        end try
    end method

    private method sendAPNS(token: String, title: String, body: String, metadata: Map): SendResult
        payload = {
            aps: {
                alert: { title: title, body: body },
                sound: "default",
                badge: metadata.get("badge") or 1
            },
            customData: metadata
        }

        try
            response = apnsClient.send(token, payload)
            return SendResult.success(response.apnsId)
        catch error
            return SendResult.failure(error.message)
        end try
    end method

    method sendBatch(notifications: List<NotificationData>): BatchResult
        results = new List()
        for each notification in notifications
            result = send(
                notification.recipient,
                notification.title,
                notification.body,
                notification.metadata
            )
            results.add(result)
        end for
        return new BatchResult(results)
    end method

    method getDeliveryStatus(messageId: String): DeliveryStatus
        // Push notifications don't typically support delivery tracking
        return DeliveryStatus.UNKNOWN
    end method

    method getCapabilities(): SenderCapabilities
        return new SenderCapabilities(
            supportsRichContent: true,
            supportsAttachments: true,
            maxBodyLength: 4096,
            supportsScheduling: false,
            supportsBatching: true
        )
    end method
end class

// Concrete Implementor: Email
class EmailNotificationSender implements NotificationSender
    private smtpClient: SMTPClient
    private templateEngine: TemplateEngine

    constructor(smtpConfig: Object)
        this.smtpClient = new SMTPClient(smtpConfig)
        this.templateEngine = new TemplateEngine()
    end constructor

    method send(
        recipient: String,
        title: String,
        body: String,
        metadata: Map
    ): SendResult
        // Build email
        email = new Email()
        email.to = recipient
        email.subject = title
        email.from = metadata.get("from") or "noreply@example.com"

        // Use template if provided
        templateName = metadata.get("template")
        if templateName != null
            email.htmlBody = templateEngine.render(templateName, {
                title: title,
                body: body,
                data: metadata
            })
            email.textBody = stripHtml(email.htmlBody)
        else
            email.textBody = body
            email.htmlBody = "<html><body>" + escapeHtml(body) + "</body></html>"
        end if

        // Handle attachments
        attachments = metadata.get("attachments")
        if attachments != null
            for each attachment in attachments
                email.addAttachment(attachment)
            end for
        end if

        try
            messageId = smtpClient.send(email)
            return SendResult.success(messageId)
        catch error
            return SendResult.failure(error.message)
        end try
    end method

    method sendBatch(notifications: List<NotificationData>): BatchResult
        // Email supports efficient batching
        results = smtpClient.sendBatch(notifications.map(n -> buildEmail(n)))
        return new BatchResult(results)
    end method

    method getDeliveryStatus(messageId: String): DeliveryStatus
        // Check bounce/delivery status if using service with tracking
        return smtpClient.getStatus(messageId)
    end method

    method getCapabilities(): SenderCapabilities
        return new SenderCapabilities(
            supportsRichContent: true,
            supportsAttachments: true,
            maxBodyLength: 1000000,  // Effectively unlimited
            supportsScheduling: true,
            supportsBatching: true
        )
    end method
end class

// Concrete Implementor: SMS
class SMSNotificationSender implements NotificationSender
    private twilioClient: TwilioClient
    private fromNumber: String

    constructor(accountSid: String, authToken: String, fromNumber: String)
        this.twilioClient = new TwilioClient(accountSid, authToken)
        this.fromNumber = fromNumber
    end constructor

    method send(
        recipient: String,
        title: String,
        body: String,
        metadata: Map
    ): SendResult
        // SMS has character limits - combine title and body smartly
        maxLength = 160  // Single SMS segment
        message = title + ": " + body

        if message.length > maxLength
            message = message.substring(0, maxLength - 3) + "..."
        end if

        try
            response = twilioClient.messages.create({
                to: recipient,
                from: fromNumber,
                body: message
            })
            return SendResult.success(response.sid)
        catch error
            return SendResult.failure(error.message)
        end try
    end method

    method sendBatch(notifications: List<NotificationData>): BatchResult
        // Twilio supports batch messaging
        messages = notifications.map(n -> {
            to: n.recipient,
            from: fromNumber,
            body: truncateForSMS(n.title + ": " + n.body)
        })

        results = twilioClient.messages.createBatch(messages)
        return new BatchResult(results)
    end method

    method getDeliveryStatus(messageId: String): DeliveryStatus
        status = twilioClient.messages.get(messageId).status
        return mapTwilioStatus(status)
    end method

    method getCapabilities(): SenderCapabilities
        return new SenderCapabilities(
            supportsRichContent: false,
            supportsAttachments: false,
            maxBodyLength: 160,
            supportsScheduling: true,
            supportsBatching: true
        )
    end method
end class

// Abstraction - notification types
abstract class Notification
    protected sender: NotificationSender
    protected recipient: String
    protected scheduledTime: DateTime

    constructor(sender: NotificationSender)
        this.sender = sender
    end constructor

    method setRecipient(recipient: String)
        this.recipient = recipient
    end method

    method schedule(time: DateTime)
        this.scheduledTime = time
    end method

    method setSender(sender: NotificationSender)
        this.sender = sender
    end method

    abstract method send(): SendResult
    abstract method getTitle(): String
    abstract method getBody(): String

    protected method buildMetadata(): Map
        return new Map()
    end method
end class

// Refined Abstraction: Alert Notification
class AlertNotification extends Notification
    private severity: AlertSeverity
    private alertTitle: String
    private alertMessage: String
    private actionUrl: String

    constructor(sender: NotificationSender, severity: AlertSeverity)
        super(sender)
        this.severity = severity
    end constructor

    method setContent(title: String, message: String)
        this.alertTitle = title
        this.alertMessage = message
    end method

    method setActionUrl(url: String)
        this.actionUrl = url
    end method

    method getTitle(): String
        prefix = ""
        switch severity
            case CRITICAL: prefix = "[CRITICAL] "
            case WARNING: prefix = "[WARNING] "
            case INFO: prefix = "[INFO] "
        end switch
        return prefix + alertTitle
    end method

    method getBody(): String
        body = alertMessage
        if actionUrl != null
            body = body + "\n\nAction required: " + actionUrl
        end if
        return body
    end method

    method send(): SendResult
        // Alerts are always high priority
        metadata = buildMetadata()
        metadata.set("priority", "high")
        metadata.set("category", "alert")
        metadata.set("severity", severity.toString())

        if actionUrl != null
            metadata.set("actionUrl", actionUrl)
        end if

        return sender.send(recipient, getTitle(), getBody(), metadata)
    end method

    protected method buildMetadata(): Map
        metadata = super.buildMetadata()
        metadata.set("template", "alert-template")
        return metadata
    end method
end class

// Refined Abstraction: Reminder Notification
class ReminderNotification extends Notification
    private reminderTitle: String
    private reminderDescription: String
    private dueDate: DateTime
    private snoozeOptions: List<Duration>

    constructor(sender: NotificationSender)
        super(sender)
        this.snoozeOptions = [Duration.minutes(15), Duration.hours(1), Duration.days(1)]
    end constructor

    method setReminder(title: String, description: String, dueDate: DateTime)
        this.reminderTitle = title
        this.reminderDescription = description
        this.dueDate = dueDate
    end method

    method setSnoozeOptions(options: List<Duration>)
        this.snoozeOptions = options
    end method

    method getTitle(): String
        return "Reminder: " + reminderTitle
    end method

    method getBody(): String
        timeUntilDue = dueDate.subtract(DateTime.now())
        urgency = ""

        if timeUntilDue.isNegative()
            urgency = "OVERDUE - "
        else if timeUntilDue.lessThan(Duration.hours(1))
            urgency = "Due soon - "
        end if

        return urgency + reminderDescription + "\nDue: " + dueDate.format("MMM d, yyyy h:mm a")
    end method

    method send(): SendResult
        metadata = buildMetadata()
        metadata.set("category", "reminder")
        metadata.set("dueDate", dueDate.toISO())
        metadata.set("snoozeOptions", snoozeOptions)

        // Check sender capabilities for actions
        capabilities = sender.getCapabilities()
        if capabilities.supportsRichContent
            metadata.set("actions", [
                { id: "complete", title: "Mark Complete" },
                { id: "snooze", title: "Snooze" }
            ])
        end if

        return sender.send(recipient, getTitle(), getBody(), metadata)
    end method
end class

// Refined Abstraction: Marketing Notification
class MarketingNotification extends Notification
    private campaign: MarketingCampaign
    private personalizedContent: Map

    constructor(sender: NotificationSender, campaign: MarketingCampaign)
        super(sender)
        this.campaign = campaign
        this.personalizedContent = new Map()
    end constructor

    method personalize(userData: UserProfile)
        personalizedContent.set("firstName", userData.firstName)
        personalizedContent.set("interests", userData.interests)
        personalizedContent.set("lastPurchase", userData.lastPurchase)
    end method

    method getTitle(): String
        title = campaign.subject
        // Simple personalization
        if personalizedContent.has("firstName")
            title = title.replace("{{name}}", personalizedContent.get("firstName"))
        end if
        return title
    end method

    method getBody(): String
        body = campaign.body
        for each key, value in personalizedContent
            body = body.replace("{{" + key + "}}", value)
        end for
        return body
    end method

    method send(): SendResult
        // Check opt-in status
        if not checkMarketingConsent(recipient)
            return SendResult.failure("User has not consented to marketing")
        end if

        metadata = buildMetadata()
        metadata.set("category", "marketing")
        metadata.set("campaignId", campaign.id)
        metadata.set("unsubscribeUrl", generateUnsubscribeUrl(recipient))
        metadata.set("template", campaign.templateName)
        metadata.set("trackingPixel", generateTrackingPixel())

        return sender.send(recipient, getTitle(), getBody(), metadata)
    end method
end class

// Usage - demonstrating the Bridge pattern's flexibility
// Same notification can be sent through different channels
pushSender = new PushNotificationSender(firebaseConfig, apnsConfig)
emailSender = new EmailNotificationSender(smtpConfig)
smsSender = new SMSNotificationSender(twilioSid, twilioToken, "+1234567890")

// Create an alert
alert = new AlertNotification(pushSender, AlertSeverity.CRITICAL)
alert.setContent("Server Down", "Production server web-01 is not responding")
alert.setActionUrl("https://dashboard.example.com/incidents/123")

// Send to multiple channels - same notification, different senders
alert.setRecipient("fcm:user_push_token")
alert.send()  // Sends push notification

alert.setSender(emailSender)
alert.setRecipient("oncall@example.com")
alert.send()  // Sends email

alert.setSender(smsSender)
alert.setRecipient("+1987654321")
alert.send()  // Sends SMS

// Create a marketing campaign notification
campaign = loadCampaign("spring-sale-2024")
promo = new MarketingNotification(emailSender, campaign)
promo.setRecipient("customer@email.com")
promo.personalize(loadUserProfile("customer@email.com"))
promo.send()

// Multi-channel notification service using Bridge
class NotificationService
    private senders: Map<String, NotificationSender>
    private userPreferences: UserPreferenceRepository

    constructor()
        senders = new Map()
        senders.set("push", new PushNotificationSender(...))
        senders.set("email", new EmailNotificationSender(...))
        senders.set("sms", new SMSNotificationSender(...))
    end constructor

    method sendToUser(userId: String, notification: Notification)
        preferences = userPreferences.getForUser(userId)
        channels = preferences.getPreferredChannels(notification.getCategory())

        for each channel in channels
            sender = senders.get(channel)
            recipient = preferences.getRecipientForChannel(channel)

            notification.setSender(sender)
            notification.setRecipient(recipient)
            notification.send()
        end for
    end method
end class
```

## Known Uses

- **Java AWT/Swing**: The AWT uses Bridge to separate window abstraction (`Window`, `Frame`) from platform-specific implementations (`WindowPeer`, `FramePeer`).

- **JDBC**: The `DriverManager` and `Connection` interface form a bridge where the abstraction (JDBC API) is separated from database-specific implementations (MySQL driver, PostgreSQL driver).

- **Device Drivers**: Operating systems use Bridge to separate the driver interface from hardware-specific implementations, allowing the same OS to work with different hardware.

- **Logging Frameworks**: SLF4J provides an abstraction layer that bridges to various logging implementations (Logback, Log4j, java.util.logging).

- **React Native**: Bridges JavaScript code (abstraction) to native platform code (implementation) on iOS and Android.

- **Graphic Libraries**: OpenGL and DirectX abstractions often use Bridge internally to support different GPU vendors.

- **Payment Processing**: Payment gateway libraries bridge merchant code to various payment processor APIs (Stripe, PayPal, Square).

- **Message Queues**: Libraries like Spring AMQP bridge application code to different message broker implementations (RabbitMQ, ActiveMQ).

## Related Patterns

- **Abstract Factory**: Can be used to create and configure a Bridge. The factory can create matched pairs of abstractions and implementors.

- **Adapter**: Both involve indirection to another object. Bridge is designed up-front to let abstraction and implementation vary independently; Adapter makes unrelated classes work together after they have been designed.

- **Strategy**: Very similar in structure to Bridge but with different intent. Strategy encapsulates algorithms; Bridge separates abstraction from implementation.

- **State**: Has the same structure as Bridge. State allows an object to change its behavior; Bridge separates a class hierarchy.

- **Template Method**: Uses inheritance to vary parts of an algorithm. Bridge uses composition to vary implementation.

- **Composite**: Can be combined with Bridge to create tree structures where nodes can have different implementations.

## When NOT to Use

- **Single implementation**: If there will only ever be one implementation, Bridge adds unnecessary abstraction. Wait until you actually need a second implementation.

- **Stable dimensions**: If neither the abstraction nor the implementation is likely to change or extend, the extra indirection provides no benefit.

- **Simple cases**: For straightforward scenarios where inheritance handles variation adequately, Bridge is overkill.

- **Performance-critical paths**: The double dispatch can add measurable overhead in tight loops. Profile before adding Bridge to hot code paths.

- **Clear abstraction boundary missing**: If you cannot clearly separate "what" from "how," forcing a Bridge will result in awkward, arbitrary splits.

- **Small class hierarchies**: If M * N is still a manageable number of classes (say, fewer than 10), the combinatorial explosion argument does not apply.

- **Tightly coupled abstraction and implementation**: Sometimes the abstraction inherently depends on implementation details. Forcing separation can lead to leaky abstractions.

- **Prototype or throwaway code**: The upfront design cost of Bridge is not worth it for code that will not be maintained long-term.

---

## Summary

The Bridge pattern is essential when you have class hierarchies that grow in two orthogonal dimensions. By splitting the monolithic hierarchy into abstraction and implementation hierarchies connected by composition, you gain flexibility, reduce class explosion, and enable runtime variation. The key insight is recognizing when you have two independent axes of change and separating them before the combinatorial growth becomes unmanageable.

---

*Based on concepts from "Design Patterns: Elements of Reusable Object-Oriented Software" by Gamma, Helm, Johnson, and Vlissides (Gang of Four), 1994.*
