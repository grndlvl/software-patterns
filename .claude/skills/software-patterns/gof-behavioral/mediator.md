# Mediator

## Intent

Define an object that encapsulates how a set of objects interact. Mediator promotes loose coupling by keeping objects from referring to each other explicitly, and it lets you vary their interaction independently.

## Also Known As

- Controller
- Coordinator

## Motivation

Consider an air traffic control system. Multiple aircraft approach an airport, each needing to coordinate with others to avoid collisions and sequence properly for landing. If each aircraft communicated directly with every other aircraft, the system would be a tangled web of connections. With N aircraft, you'd have N*(N-1)/2 potential communication channels, each with its own protocol and state to manage.

The solution is air traffic control—a mediator. Aircraft don't communicate with each other directly; they communicate only with the control tower. The tower has complete visibility of all aircraft, their positions, speeds, and intentions. It makes decisions about sequencing, spacing, and runway assignments, then communicates instructions to individual aircraft.

This mediator pattern dramatically simplifies the system. Aircraft only need to know how to talk to the tower, not to every other aircraft. Adding a new aircraft doesn't require updating all existing aircraft. The coordination logic lives in one place (the tower) rather than being scattered across all participants. And if coordination rules change (new approach patterns, different spacing requirements), only the tower needs updating.

The same principle applies in software. GUI dialog boxes, chat rooms, workflow engines, and microservice orchestrators all benefit from centralizing coordination logic in a mediator rather than distributing it across many interconnected objects.

## Applicability

Use the Mediator pattern when:

- A set of objects communicate in well-defined but complex ways. The resulting interdependencies are unstructured and difficult to understand.
- Reusing an object is difficult because it refers to and communicates with many other objects.
- A behavior that's distributed between several classes should be customizable without a lot of subclassing.
- You want to decouple components so they can be developed, tested, and modified independently.
- The interaction logic is complex enough that centralizing it improves clarity.

Common applications include:
- GUI dialog boxes coordinating widgets
- Chat rooms coordinating participants
- Air traffic control systems
- Workflow engines coordinating activities
- Game engines coordinating game objects
- Microservice orchestrators
- Event buses and message brokers

## Structure

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│                  ┌───────────────────────────────────────────┐              │
│                  │            <<interface>>                  │              │
│                  │              Mediator                     │              │
│                  ├───────────────────────────────────────────┤              │
│                  │ + notify(sender: Colleague, event): void  │              │
│                  └───────────────────────────────────────────┘              │
│                                      △                                      │
│                                      │                                      │
│                  ┌───────────────────┴───────────────────────┐              │
│                  │          ConcreteMediator                 │              │
│                  ├───────────────────────────────────────────┤              │
│                  │ - colleagueA: ColleagueA                  │              │
│                  │ - colleagueB: ColleagueB                  │              │
│                  │ - colleagueC: ColleagueC                  │              │
│                  ├───────────────────────────────────────────┤              │
│                  │ + notify(sender, event)                   │              │
│                  │ + registerColleague(colleague)            │              │
│                  └───────────────────────────────────────────┘              │
│                           │           │           │                         │
│                           │  knows    │           │                         │
│              ┌────────────┘           │           └────────────┐            │
│              │                        │                        │            │
│              ▼                        ▼                        ▼            │
│    ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐     │
│    │   ColleagueA    │     │   ColleagueB    │     │   ColleagueC    │     │
│    ├─────────────────┤     ├─────────────────┤     ├─────────────────┤     │
│    │-mediator        │     │-mediator        │     │-mediator        │     │
│    ├─────────────────┤     ├─────────────────┤     ├─────────────────┤     │
│    │+setMediator()   │     │+setMediator()   │     │+setMediator()   │     │
│    │+operation()     │     │+operation()     │     │+operation()     │     │
│    └─────────────────┘     └─────────────────┘     └─────────────────┘     │
│              │                        │                        │            │
│              └────────────────────────┼────────────────────────┘            │
│                                       │                                     │
│                              notifies │                                     │
│                                       ▼                                     │
│                               ┌───────────────┐                             │
│                               │   Mediator    │                             │
│                               └───────────────┘                             │
│                                                                             │
│   Before Mediator:                    After Mediator:                       │
│                                                                             │
│       A ←──→ B                            A                                 │
│       ↕  ╲╱  ↕                             ↘                               │
│       D ←──→ C                        D ──→ M ←── B                        │
│                                             ↗                               │
│   (Many-to-many)                          C                                │
│                                       (Star topology)                       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Participants

- **Mediator**: Defines an interface for communicating with Colleague objects. Typically includes a `notify()` method that colleagues call when events occur.

- **ConcreteMediator**: Implements cooperative behavior by coordinating Colleague objects. Knows and maintains references to its colleagues. Contains the interaction logic that would otherwise be distributed among colleagues.

- **Colleague**: Each Colleague class knows its Mediator object. Colleagues communicate with the mediator whenever they would have otherwise communicated with another colleague.

- **ConcreteColleague**: Each concrete colleague communicates with its mediator when it needs to communicate with other colleagues. Colleagues are often unaware of each other's existence.

## Collaborations

1. Colleagues send and receive requests from a Mediator object. The mediator implements the cooperative behavior by routing requests between appropriate colleagues.

2. When a colleague's state changes or an event occurs, it notifies the mediator.

3. The mediator responds by calling appropriate methods on other colleagues, coordinating their behavior.

4. Colleagues don't communicate directly with each other—all communication goes through the mediator.

## Consequences

### Benefits

1. **Limits subclassing**: A mediator localizes behavior that otherwise would be distributed among several objects. Changing this behavior requires subclassing the mediator only; colleague classes can be reused as is.

2. **Decouples colleagues**: Colleagues become independent of each other. You can vary and reuse colleague classes independently of the mediator. Adding new colleagues requires changing only the mediator.

3. **Simplifies object protocols**: A mediator replaces many-to-many interactions with one-to-many interactions between the mediator and its colleagues. One-to-many relationships are easier to understand, maintain, and extend.

4. **Abstracts how objects cooperate**: Making mediation an independent concept and encapsulating it in an object lets you focus on how objects interact apart from their individual behavior.

5. **Centralizes control**: The mediator centralizes control logic in one place, making it easier to comprehend how the system's objects interact.

### Liabilities

1. **Mediator complexity**: The mediator can become a monolith that's hard to maintain. If it grows too complex, it may need to be decomposed into smaller mediators.

2. **Single point of failure**: All communication depends on the mediator. If it fails or becomes a bottleneck, the entire system suffers.

3. **Can reduce reusability of colleagues**: Colleagues that are tightly coupled to a specific mediator interface may be harder to reuse in other contexts.

## Implementation

### Implementation Considerations

1. **Omitting the abstract Mediator class**: When colleagues work with only one mediator, there's no need for an abstract Mediator class. The abstract coupling provides flexibility when colleagues may work with different mediators.

2. **Colleague-Mediator communication**: Colleagues can communicate with the mediator in several ways:
   - Observer pattern: Colleagues publish events; mediator subscribes
   - Direct method calls: Colleagues call mediator methods
   - Command objects: Colleagues send command objects to mediator

3. **Initialization**: The mediator needs references to colleagues, and colleagues need a reference to the mediator. This can be handled via constructor injection, setter methods, or a registration mechanism.

### Pseudocode: Mediator Infrastructure

```
// Mediator interface
interface Mediator {
    method notify(sender: Component, event: string, data: any): void
}

// Base colleague class
abstract class Component {
    protected mediator: Mediator

    method setMediator(mediator: Mediator): void {
        this.mediator = mediator
    }

    protected method notifyMediator(event: string, data: any = null): void {
        if this.mediator is not null {
            this.mediator.notify(this, event, data)
        }
    }
}
```

### Pseudocode: Dialog Box Example

```
// Concrete mediator for a user registration dialog
class RegistrationDialog implements Mediator {
    private usernameInput: TextInput
    private emailInput: TextInput
    private passwordInput: TextInput
    private confirmPasswordInput: TextInput
    private termsCheckbox: Checkbox
    private submitButton: Button
    private statusLabel: Label

    constructor() {
        // Create all components
        this.usernameInput = new TextInput("username")
        this.emailInput = new TextInput("email")
        this.passwordInput = new TextInput("password", masked: true)
        this.confirmPasswordInput = new TextInput("confirmPassword", masked: true)
        this.termsCheckbox = new Checkbox("terms")
        this.submitButton = new Button("submit", "Register")
        this.statusLabel = new Label("status")

        // Register this mediator with all components
        this.usernameInput.setMediator(this)
        this.emailInput.setMediator(this)
        this.passwordInput.setMediator(this)
        this.confirmPasswordInput.setMediator(this)
        this.termsCheckbox.setMediator(this)
        this.submitButton.setMediator(this)
        this.statusLabel.setMediator(this)

        // Initial state
        this.submitButton.setEnabled(false)
    }

    method notify(sender: Component, event: string, data: any): void {
        switch event {
            case "textChanged":
                this.validateForm()
                break

            case "checkChanged":
                this.validateForm()
                break

            case "buttonClicked":
                if sender == this.submitButton {
                    this.handleSubmit()
                }
                break

            case "focus":
                this.clearStatus()
                break
        }
    }

    private method validateForm(): void {
        errors = []

        // Validate username
        username = this.usernameInput.getValue()
        if username.length < 3 {
            errors.add("Username must be at least 3 characters")
        }

        // Validate email
        email = this.emailInput.getValue()
        if not isValidEmail(email) {
            errors.add("Invalid email address")
        }

        // Validate password
        password = this.passwordInput.getValue()
        if password.length < 8 {
            errors.add("Password must be at least 8 characters")
        }

        // Validate password confirmation
        confirmPassword = this.confirmPasswordInput.getValue()
        if password != confirmPassword {
            errors.add("Passwords don't match")
            this.confirmPasswordInput.setError(true)
        } else {
            this.confirmPasswordInput.setError(false)
        }

        // Validate terms checkbox
        termsAccepted = this.termsCheckbox.isChecked()
        if not termsAccepted {
            errors.add("You must accept the terms")
        }

        // Update submit button state
        this.submitButton.setEnabled(errors.isEmpty())

        // Update status
        if errors.isNotEmpty() {
            this.statusLabel.setText(errors.first())
            this.statusLabel.setStyle("error")
        } else {
            this.statusLabel.setText("Ready to register")
            this.statusLabel.setStyle("success")
        }
    }

    private method handleSubmit(): void {
        this.submitButton.setEnabled(false)
        this.statusLabel.setText("Registering...")
        this.statusLabel.setStyle("info")

        // Collect form data
        formData = {
            username: this.usernameInput.getValue(),
            email: this.emailInput.getValue(),
            password: this.passwordInput.getValue()
        }

        // Submit (in real code, this would be async)
        try {
            result = this.submitRegistration(formData)
            this.statusLabel.setText("Registration successful!")
            this.statusLabel.setStyle("success")
            this.clearForm()
        } catch error {
            this.statusLabel.setText("Registration failed: " + error.message)
            this.statusLabel.setStyle("error")
            this.submitButton.setEnabled(true)
        }
    }

    private method clearForm(): void {
        this.usernameInput.setValue("")
        this.emailInput.setValue("")
        this.passwordInput.setValue("")
        this.confirmPasswordInput.setValue("")
        this.termsCheckbox.setChecked(false)
    }

    private method clearStatus(): void {
        // Don't clear if showing success/error from submission
    }

    private method submitRegistration(data: object): object {
        // API call
    }
}

// Colleague: Text input component
class TextInput extends Component {
    private name: string
    private value: string = ""
    private masked: boolean
    private hasError: boolean = false

    constructor(name: string, masked: boolean = false) {
        this.name = name
        this.masked = masked
    }

    method getValue(): string {
        return this.value
    }

    method setValue(value: string): void {
        this.value = value
        this.notifyMediator("textChanged", this.value)
    }

    method setError(hasError: boolean): void {
        this.hasError = hasError
        // Update visual appearance
    }

    method onUserInput(newValue: string): void {
        this.value = newValue
        this.notifyMediator("textChanged", this.value)
    }

    method onFocus(): void {
        this.notifyMediator("focus", null)
    }
}

// Colleague: Checkbox component
class Checkbox extends Component {
    private name: string
    private checked: boolean = false

    constructor(name: string) {
        this.name = name
    }

    method isChecked(): boolean {
        return this.checked
    }

    method setChecked(checked: boolean): void {
        this.checked = checked
        this.notifyMediator("checkChanged", this.checked)
    }

    method onUserClick(): void {
        this.checked = not this.checked
        this.notifyMediator("checkChanged", this.checked)
    }
}

// Colleague: Button component
class Button extends Component {
    private name: string
    private label: string
    private enabled: boolean = true

    constructor(name: string, label: string) {
        this.name = name
        this.label = label
    }

    method setEnabled(enabled: boolean): void {
        this.enabled = enabled
        // Update visual appearance
    }

    method isEnabled(): boolean {
        return this.enabled
    }

    method onUserClick(): void {
        if this.enabled {
            this.notifyMediator("buttonClicked", null)
        }
    }
}

// Colleague: Label component
class Label extends Component {
    private name: string
    private text: string = ""
    private style: string = "normal"

    constructor(name: string) {
        this.name = name
    }

    method setText(text: string): void {
        this.text = text
        // Update display
    }

    method setStyle(style: string): void {
        this.style = style
        // Update visual appearance ("normal", "error", "success", "info")
    }
}
```

## Example

A complete example implementing a chat room mediator:

```
// Message types
enum MessageType {
    PUBLIC,
    PRIVATE,
    SYSTEM
}

class Message {
    public type: MessageType
    public sender: string
    public content: string
    public recipient: string = null  // For private messages
    public timestamp: datetime

    constructor(type: MessageType, sender: string, content: string) {
        this.type = type
        this.sender = sender
        this.content = content
        this.timestamp = now()
    }

    static method publicMessage(sender: string, content: string): Message {
        return new Message(MessageType.PUBLIC, sender, content)
    }

    static method privateMessage(sender: string, recipient: string, content: string): Message {
        msg = new Message(MessageType.PRIVATE, sender, content)
        msg.recipient = recipient
        return msg
    }

    static method systemMessage(content: string): Message {
        return new Message(MessageType.SYSTEM, "System", content)
    }
}

// Chat room mediator interface
interface ChatMediator {
    method register(participant: ChatParticipant): void
    method unregister(participant: ChatParticipant): void
    method sendMessage(message: Message): void
    method getParticipants(): list<string>
}

// Concrete chat room mediator
class ChatRoom implements ChatMediator {
    private name: string
    private participants: map<string, ChatParticipant> = {}
    private messageHistory: list<Message> = []
    private maxHistorySize: int = 100
    private bannedUsers: set<string> = {}
    private mutedUsers: map<string, datetime> = {}  // User -> mute expiry

    constructor(name: string) {
        this.name = name
    }

    method register(participant: ChatParticipant): void {
        username = participant.getUsername()

        if this.bannedUsers.contains(username) {
            participant.receive(Message.systemMessage("You are banned from this room"))
            return
        }

        if this.participants.containsKey(username) {
            participant.receive(Message.systemMessage("Username already taken"))
            return
        }

        this.participants.set(username, participant)
        participant.setMediator(this)

        // Send recent history to new participant
        recentMessages = this.getRecentHistory(10)
        for msg in recentMessages {
            participant.receive(msg)
        }

        // Announce new participant
        announcement = Message.systemMessage(username + " has joined the chat")
        this.broadcast(announcement, exclude: username)
    }

    method unregister(participant: ChatParticipant): void {
        username = participant.getUsername()

        if this.participants.containsKey(username) {
            this.participants.remove(username)

            // Announce departure
            announcement = Message.systemMessage(username + " has left the chat")
            this.broadcast(announcement)
        }
    }

    method sendMessage(message: Message): void {
        sender = message.sender

        // Check if sender is muted
        if this.isUserMuted(sender) {
            senderParticipant = this.participants.get(sender)
            if senderParticipant is not null {
                senderParticipant.receive(Message.systemMessage("You are muted"))
            }
            return
        }

        // Store in history
        this.addToHistory(message)

        // Route message
        if message.type == MessageType.PRIVATE {
            this.routePrivateMessage(message)
        } else {
            this.broadcast(message)
        }
    }

    method getParticipants(): list<string> {
        return this.participants.keys().toList()
    }

    private method broadcast(message: Message, exclude: string = null): void {
        for username, participant in this.participants {
            if username != exclude {
                participant.receive(message)
            }
        }
    }

    private method routePrivateMessage(message: Message): void {
        recipient = this.participants.get(message.recipient)
        sender = this.participants.get(message.sender)

        if recipient is null {
            if sender is not null {
                sender.receive(Message.systemMessage(
                    "User '" + message.recipient + "' not found"
                ))
            }
            return
        }

        // Send to recipient
        recipient.receive(message)

        // Send confirmation to sender
        if sender is not null and sender != recipient {
            sender.receive(message)  // Echo the private message
        }
    }

    private method addToHistory(message: Message): void {
        this.messageHistory.add(message)

        // Trim history if too large
        while this.messageHistory.size() > this.maxHistorySize {
            this.messageHistory.removeFirst()
        }
    }

    private method getRecentHistory(count: int): list<Message> {
        start = max(0, this.messageHistory.size() - count)
        return this.messageHistory.subList(start, this.messageHistory.size())
    }

    private method isUserMuted(username: string): boolean {
        if not this.mutedUsers.containsKey(username) {
            return false
        }

        muteExpiry = this.mutedUsers.get(username)
        if now() > muteExpiry {
            this.mutedUsers.remove(username)
            return false
        }

        return true
    }

    // Moderation methods
    method banUser(username: string, moderator: string): void {
        if this.participants.containsKey(username) {
            participant = this.participants.get(username)
            participant.receive(Message.systemMessage(
                "You have been banned by " + moderator
            ))
            this.unregister(participant)
        }

        this.bannedUsers.add(username)
        this.broadcast(Message.systemMessage(
            username + " has been banned by " + moderator
        ))
    }

    method muteUser(username: string, durationMinutes: int, moderator: string): void {
        expiry = now().plusMinutes(durationMinutes)
        this.mutedUsers.set(username, expiry)

        participant = this.participants.get(username)
        if participant is not null {
            participant.receive(Message.systemMessage(
                "You have been muted for " + durationMinutes + " minutes"
            ))
        }

        this.broadcast(Message.systemMessage(
            username + " has been muted by " + moderator
        ))
    }
}

// Abstract chat participant (colleague)
abstract class ChatParticipant {
    protected username: string
    protected mediator: ChatMediator

    constructor(username: string) {
        this.username = username
    }

    method getUsername(): string {
        return this.username
    }

    method setMediator(mediator: ChatMediator): void {
        this.mediator = mediator
    }

    method send(content: string): void {
        if this.mediator is null {
            return
        }
        message = Message.publicMessage(this.username, content)
        this.mediator.sendMessage(message)
    }

    method sendPrivate(recipient: string, content: string): void {
        if this.mediator is null {
            return
        }
        message = Message.privateMessage(this.username, recipient, content)
        this.mediator.sendMessage(message)
    }

    method leave(): void {
        if this.mediator is not null {
            this.mediator.unregister(this)
            this.mediator = null
        }
    }

    method listUsers(): list<string> {
        if this.mediator is null {
            return []
        }
        return this.mediator.getParticipants()
    }

    abstract method receive(message: Message): void
}

// Concrete participant: Regular user with console output
class ConsoleUser extends ChatParticipant {
    constructor(username: string) {
        super(username)
    }

    method receive(message: Message): void {
        timestamp = formatTime(message.timestamp)

        switch message.type {
            case MessageType.SYSTEM:
                print("[" + timestamp + "] *** " + message.content + " ***")
                break

            case MessageType.PRIVATE:
                if message.sender == this.username {
                    print("[" + timestamp + "] (to " + message.recipient + ") " +
                          message.content)
                } else {
                    print("[" + timestamp + "] (private from " + message.sender + ") " +
                          message.content)
                }
                break

            case MessageType.PUBLIC:
                print("[" + timestamp + "] " + message.sender + ": " + message.content)
                break
        }
    }
}

// Concrete participant: Bot that responds to commands
class ChatBot extends ChatParticipant {
    private commands: map<string, function(list<string>): string>

    constructor(username: string) {
        super(username)
        this.setupCommands()
    }

    private method setupCommands(): void {
        this.commands = {}

        this.commands.set("help", (args) => {
            return "Available commands: !help, !time, !users, !flip, !roll [sides]"
        })

        this.commands.set("time", (args) => {
            return "Current time: " + formatDateTime(now())
        })

        this.commands.set("users", (args) => {
            users = this.listUsers()
            return "Online users: " + join(users, ", ")
        })

        this.commands.set("flip", (args) => {
            result = random() < 0.5 ? "heads" : "tails"
            return "Coin flip: " + result
        })

        this.commands.set("roll", (args) => {
            sides = 6
            if args.size() > 0 {
                sides = parseInt(args.get(0), default: 6)
            }
            result = randomInt(1, sides)
            return "Rolled d" + sides + ": " + result
        })
    }

    method receive(message: Message): void {
        // Ignore system messages and own messages
        if message.type == MessageType.SYSTEM {
            return
        }
        if message.sender == this.username {
            return
        }

        content = message.content.trim()

        // Check for command
        if content.startsWith("!") {
            this.handleCommand(message.sender, content)
        }
    }

    private method handleCommand(sender: string, content: string): void {
        parts = content.substring(1).split(" ")
        command = parts.get(0).toLowerCase()
        args = parts.subList(1, parts.size())

        handler = this.commands.get(command)
        if handler is not null {
            response = handler(args)
            this.send("@" + sender + " " + response)
        }
    }
}

// Concrete participant: Logging participant that archives messages
class MessageLogger extends ChatParticipant {
    private logFile: string
    private logStream: FileOutputStream

    constructor(logFile: string) {
        super("Logger")
        this.logFile = logFile
        this.logStream = new FileOutputStream(logFile, append: true)
    }

    method receive(message: Message): void {
        logEntry = formatLogEntry(message)
        this.logStream.write(logEntry + "\n")
        this.logStream.flush()
    }

    private method formatLogEntry(message: Message): string {
        return JSON.stringify({
            timestamp: message.timestamp.toISOString(),
            type: message.type.name,
            sender: message.sender,
            recipient: message.recipient,
            content: message.content
        })
    }

    method close(): void {
        this.leave()
        this.logStream.close()
    }
}

// Moderated chat room with additional controls
class ModeratedChatRoom extends ChatRoom {
    private moderators: set<string> = {}
    private wordFilter: list<string> = []

    constructor(name: string) {
        super(name)
    }

    method addModerator(username: string): void {
        this.moderators.add(username)
    }

    method isModerator(username: string): boolean {
        return this.moderators.contains(username)
    }

    method setWordFilter(words: list<string>): void {
        this.wordFilter = words.map(w => w.toLowerCase())
    }

    override method sendMessage(message: Message): void {
        // Filter content
        if this.containsFilteredWord(message.content) {
            sender = this.participants.get(message.sender)
            if sender is not null {
                sender.receive(Message.systemMessage(
                    "Your message was blocked by the word filter"
                ))
            }
            return
        }

        super.sendMessage(message)
    }

    private method containsFilteredWord(content: string): boolean {
        lowerContent = content.toLowerCase()
        for word in this.wordFilter {
            if lowerContent.contains(word) {
                return true
            }
        }
        return false
    }
}

// Usage example
function main() {
    // Create chat room
    chatRoom = new ModeratedChatRoom("General")
    chatRoom.setWordFilter(["spam", "scam"])

    // Create participants
    alice = new ConsoleUser("Alice")
    bob = new ConsoleUser("Bob")
    charlie = new ConsoleUser("Charlie")
    bot = new ChatBot("HelpBot")
    logger = new MessageLogger("/var/log/chat.log")

    // Register participants (they automatically get the mediator reference)
    chatRoom.register(alice)
    chatRoom.register(bob)
    chatRoom.register(charlie)
    chatRoom.register(bot)
    chatRoom.register(logger)

    // Add moderator
    chatRoom.addModerator("Alice")

    // Simulate conversation
    alice.send("Hello everyone!")
    // Output for Bob, Charlie: [10:30:00] Alice: Hello everyone!

    bob.send("Hi Alice!")
    // Output for Alice, Charlie: [10:30:01] Bob: Hi Alice!

    charlie.sendPrivate("Alice", "Can we talk privately?")
    // Output for Alice: [10:30:02] (private from Charlie) Can we talk privately?
    // Output for Charlie: [10:30:02] (to Alice) Can we talk privately?

    bob.send("!help")
    // Bot responds: [10:30:03] HelpBot: @Bob Available commands: !help, !time...

    bob.send("!roll 20")
    // Bot responds: [10:30:04] HelpBot: @Bob Rolled d20: 17

    // List online users
    print("Online: " + join(alice.listUsers(), ", "))
    // Output: Online: Alice, Bob, Charlie, HelpBot, Logger

    // Moderation
    chatRoom.muteUser("Charlie", 5, "Alice")
    charlie.send("I'm muted!")
    // Charlie receives: [10:30:05] *** You are muted ***

    // User leaves
    bob.leave()
    // Everyone receives: [10:30:06] *** Bob has left the chat ***

    // Cleanup
    logger.close()
}
```

## Known Uses

- **Java AWT/Swing**: The `java.awt.Dialog` class acts as a mediator for its components.

- **Spring MVC**: The DispatcherServlet mediates between HTTP requests and controllers/views.

- **Ruby on Rails**: ActiveRecord callbacks and observers create implicit mediator relationships.

- **Redux**: The store mediates between action dispatchers and state subscribers.

- **Message Brokers**: RabbitMQ, Apache Kafka, and similar systems mediate between message producers and consumers.

- **Air Traffic Control Systems**: Real-world ATC systems are canonical examples of the Mediator pattern.

- **Chat Applications**: IRC servers, Slack, Discord all use mediator-style architecture for message routing.

- **MVVM Frameworks**: The ViewModel often acts as a mediator between View and Model.

- **Workflow Engines**: BPMN engines, Apache Airflow, and similar tools mediate between workflow activities.

## Related Patterns

- **Facade**: Both abstract functionality of existing classes. Facade defines a simpler interface to a subsystem; Mediator abstracts communication between colleagues. Facade is unidirectional; Mediator enables multidirectional communication.

- **Observer**: Colleagues can communicate with the mediator using the Observer pattern. The mediator acts as an observer of colleague events.

- **Command**: Colleagues can encapsulate requests in Command objects that they send to the mediator.

- **Singleton**: Mediators are often singletons since there's typically only one mediator for a set of colleagues.

## When NOT to Use

1. **Few components with simple interactions**: If you only have 2-3 objects with simple interactions, direct communication is clearer.

```
// Overkill: Mediator for two objects
class TwoButtonMediator {
    method notify(sender, event) {
        if sender == buttonA and event == "click" {
            buttonB.setEnabled(false)
        }
    }
}

// Just do this:
buttonA.onClick = () => buttonB.setEnabled(false)
```

2. **Interactions that rarely change**: If the interaction logic is stable and well-understood, the indirection of a mediator adds complexity without benefit.

3. **When colleagues need direct references anyway**: If colleagues need to call specific methods on each other (not just notify of events), the mediator becomes a pass-through that adds no value.

4. **Performance-critical communication**: The indirection through a mediator adds overhead. For high-frequency, low-latency communication, direct calls may be necessary.

5. **When it creates a God object**: If your mediator knows everything about every colleague and contains all business logic, you've created a maintenance nightmare. Consider decomposing into smaller mediators or using other patterns.

```
// Anti-pattern: God mediator
class EverythingMediator {
    method notify(sender, event) {
        // 500 lines of conditional logic handling
        // every possible interaction between dozens of objects
        // This is worse than distributed coupling!
    }
}
```

6. **Stateless request routing**: If you're just routing messages without coordination logic, a simple event bus or message queue may be more appropriate.

The Mediator pattern excels when you have many objects with complex, changing interactions. It's overkill for simple scenarios and becomes a liability if the mediator itself becomes too complex. Use it when the benefit of centralized coordination outweighs the cost of the indirection.

---

*Based on concepts from "Design Patterns: Elements of Reusable Object-Oriented Software" by Gamma, Helm, Johnson, and Vlissides (Gang of Four), 1994.*
