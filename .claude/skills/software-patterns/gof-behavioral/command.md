# Command

## Intent

Encapsulate a request as an object, thereby letting you parameterize clients with different requests, queue or log requests, and support undoable operations.

## Also Known As

- Action
- Transaction
- Operation

## Motivation

Consider a text editor with a rich user interface featuring toolbars, menus, and keyboard shortcuts. Each of these UI elements can trigger operations like copy, paste, bold, italic, save, and undo. The naive approach would have each button or menu item directly call the appropriate method on the document object. However, this creates tight coupling and makes it impossible to implement features like undo/redo, macro recording, or operation queuing.

The Command pattern solves this by turning requests into stand-alone objects that contain all information about the request. Instead of a toolbar button directly calling `document.bold()`, it creates a `BoldCommand` object and executes it. This command object knows how to perform the operation AND how to undo it.

This transformation provides remarkable flexibility. Commands can be stored in a history stack for undo/redo functionality. They can be serialized and sent over a network for distributed systems. They can be queued for batch processing or scheduled execution. Multiple commands can be composed into macro commands. And because commands are objects, they can be passed around, stored, and manipulated just like any other data.

The decoupling between the invoker (button, menu item) and the receiver (document) means you can change either without affecting the other. You can add new commands without modifying existing UI code, and you can reuse the same command across multiple UI elements (menu, toolbar, shortcut all share the same CopyCommand).

## Applicability

Use the Command pattern when:

- You want to parameterize objects with an action to perform. Commands are an object-oriented replacement for callbacks.
- You need to specify, queue, and execute requests at different times. A command object can have a lifetime independent of the original request.
- You need to support undo. The Command's execute operation can store state for reversing its effects. The Command interface must include an unexecute operation.
- You need to support logging changes so they can be reapplied after a system crash. By augmenting the Command interface with load and store operations, you can keep a persistent log of changes.
- You want to structure a system around high-level operations built on primitive operations. This is common in transaction-based systems.
- You need to implement macro recording by storing a sequence of commands.
- You want to decouple the object that invokes the operation from the one that performs it.

## Structure

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│    ┌────────┐         ┌───────────────────────────────────────────┐        │
│    │ Client │────────>│              <<interface>>                │        │
│    └────────┘         │                Command                    │        │
│         │             ├───────────────────────────────────────────┤        │
│         │             │ + execute(): void                         │        │
│         │             │ + undo(): void                            │        │
│         │             │ + redo(): void                            │        │
│         │             └───────────────────────────────────────────┘        │
│         │                               △                                   │
│         │                               │                                   │
│         │             ┌─────────────────┴─────────────────────┐            │
│         │             │                                       │            │
│         │    ┌────────┴────────────┐            ┌────────────┴──────────┐ │
│         │    │  ConcreteCommandA   │            │   ConcreteCommandB    │ │
│         │    ├─────────────────────┤            ├───────────────────────┤ │
│         │    │ - receiver: Receiver│            │ - receiver: Receiver  │ │
│         │    │ - state: any        │            │ - params: any         │ │
│         │    ├─────────────────────┤            ├───────────────────────┤ │
│         │    │ + execute()         │            │ + execute()           │ │
│         │    │ + undo()            │            │ + undo()              │ │
│         │    └─────────────────────┘            └───────────────────────┘ │
│         │              │                                   │               │
│         │              │ calls                             │ calls         │
│         ▼              ▼                                   ▼               │
│    ┌─────────┐    ┌──────────┐                                            │
│    │ Invoker │    │ Receiver │                                            │
│    ├─────────┤    ├──────────┤                                            │
│    │-command │    │ +action()│                                            │
│    ├─────────┤    │ +state   │                                            │
│    │+setCmd()│    └──────────┘                                            │
│    │+invoke()│                                                             │
│    └─────────┘                                                             │
│                                                                             │
│   Sequence:                                                                 │
│   ┌────────┐      ┌─────────┐      ┌─────────┐      ┌──────────┐          │
│   │ Client │      │ Invoker │      │ Command │      │ Receiver │          │
│   └───┬────┘      └────┬────┘      └────┬────┘      └────┬─────┘          │
│       │                │                │                │                 │
│       │ setCommand(c)  │                │                │                 │
│       │───────────────>│                │                │                 │
│       │                │                │                │                 │
│       │                │  execute()     │                │                 │
│       │                │───────────────>│                │                 │
│       │                │                │    action()    │                 │
│       │                │                │───────────────>│                 │
│       │                │                │                │                 │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Participants

- **Command**: Declares an interface for executing an operation. Typically includes `execute()`, and optionally `undo()` and `redo()` methods.

- **ConcreteCommand**: Defines a binding between a Receiver object and an action. Implements `execute()` by invoking the corresponding operation(s) on the Receiver. Stores any state needed for undo operations.

- **Client**: Creates a ConcreteCommand object and sets its receiver. The client is responsible for creating commands and configuring them with the appropriate receivers.

- **Invoker**: Asks the command to carry out the request. Stores commands and can maintain command history for undo/redo. Examples: buttons, menu items, schedulers.

- **Receiver**: Knows how to perform the operations associated with carrying out a request. Any class may serve as a Receiver. The receiver has the actual business logic.

## Collaborations

1. The client creates a ConcreteCommand object and specifies its receiver.

2. An Invoker object stores the ConcreteCommand object.

3. The invoker issues a request by calling `execute()` on the command. When commands are undoable, ConcreteCommand stores state for undoing the command prior to invoking `execute()`.

4. The ConcreteCommand object invokes operations on its receiver to carry out the request.

5. For undo functionality, the invoker maintains a history of executed commands. Calling `undo()` reverses the command's effects using the stored state.

## Consequences

### Benefits

1. **Decoupling**: Command decouples the object that invokes the operation from the one that knows how to perform it. The invoker doesn't need to know anything about what the command does or who receives it.

2. **Commands are first-class objects**: They can be manipulated and extended like any other object. You can compose them, store them, pass them as parameters.

3. **Easy to add new commands**: You can add new commands without changing existing classes. Just create a new ConcreteCommand class.

4. **Support for undo/redo**: By storing state before execution, commands can reverse their effects. A history stack enables multi-level undo.

5. **Support for transactions**: Commands can be grouped into composite commands (macros) that execute atomically. If one fails, all can be rolled back.

6. **Support for logging and persistence**: Commands can be serialized and logged. After a crash, commands can be reloaded and re-executed to restore state.

7. **Support for queuing**: Commands can be queued for later execution, enabling deferred operations, batch processing, and scheduling.

### Liabilities

1. **Increased number of classes**: Every operation needs its own ConcreteCommand class, which can lead to class proliferation.

2. **Memory overhead**: Storing commands for undo history consumes memory, especially if commands store significant state.

3. **Complexity for simple operations**: For operations that don't need undo/redo or queuing, the Command pattern adds unnecessary indirection.

4. **State management complexity**: Implementing proper undo requires careful management of command state, especially when commands interact with each other.

## Implementation

### Implementation Considerations

1. **How smart should a command be?**: Commands can range from "dumb" (just invoke a single method on the receiver) to "smart" (implement the logic themselves without a receiver). The right balance depends on your needs.

2. **Supporting undo and redo**: To support undo, commands must store enough state to reverse their effects. This includes:
   - The receiver object
   - Arguments to the operation performed on the receiver
   - Any original values in the receiver that changed (for restoration)

3. **Avoiding error accumulation in undo**: If commands are repeatedly done and undone, errors can accumulate. Consider using Memento pattern to store and restore receiver state.

4. **Using command history**: Maintain a history list of executed commands. Use a current position pointer that moves backward for undo and forward for redo.

### Pseudocode: Basic Command Infrastructure

```
// Command interface
interface Command {
    method execute(): void
    method undo(): void
    method redo(): void
    method getDescription(): string
}

// Abstract base class for commands that modify state
abstract class UndoableCommand implements Command {
    protected receiver: any
    protected previousState: any
    protected executed: boolean = false

    method redo(): void {
        // Default redo is just execute again
        this.execute()
    }

    method canUndo(): boolean {
        return this.executed
    }
}
```

### Pseudocode: Document Editor Commands

```
// Receiver: The document being edited
class TextDocument {
    private content: string = ""
    private selectionStart: int = 0
    private selectionEnd: int = 0
    private clipboard: string = ""
    private formatting: map<Range, FormatStyle> = {}

    method getContent(): string {
        return this.content
    }

    method setContent(content: string): void {
        this.content = content
    }

    method getSelection(): Range {
        return new Range(this.selectionStart, this.selectionEnd)
    }

    method getSelectedText(): string {
        return this.content.substring(this.selectionStart, this.selectionEnd)
    }

    method insertAt(position: int, text: string): void {
        this.content = this.content.substring(0, position) +
                       text +
                       this.content.substring(position)
    }

    method deleteRange(start: int, end: int): string {
        deleted = this.content.substring(start, end)
        this.content = this.content.substring(0, start) +
                       this.content.substring(end)
        return deleted
    }

    method setClipboard(text: string): void {
        this.clipboard = text
    }

    method getClipboard(): string {
        return this.clipboard
    }

    method applyFormatting(range: Range, style: FormatStyle): void {
        this.formatting.set(range, style)
    }

    method removeFormatting(range: Range): FormatStyle {
        style = this.formatting.get(range)
        this.formatting.remove(range)
        return style
    }
}

// Concrete Command: Insert Text
class InsertTextCommand extends UndoableCommand {
    private document: TextDocument
    private position: int
    private text: string

    constructor(document: TextDocument, position: int, text: string) {
        this.document = document
        this.position = position
        this.text = text
    }

    method execute(): void {
        this.document.insertAt(this.position, this.text)
        this.executed = true
    }

    method undo(): void {
        // Remove the inserted text
        this.document.deleteRange(this.position, this.position + this.text.length)
        this.executed = false
    }

    method getDescription(): string {
        preview = this.text.substring(0, min(20, this.text.length))
        if this.text.length > 20 {
            preview = preview + "..."
        }
        return "Insert: \"" + preview + "\""
    }
}

// Concrete Command: Delete Text
class DeleteTextCommand extends UndoableCommand {
    private document: TextDocument
    private start: int
    private end: int
    private deletedText: string = ""

    constructor(document: TextDocument, start: int, end: int) {
        this.document = document
        this.start = start
        this.end = end
    }

    method execute(): void {
        // Store the text before deleting (for undo)
        this.deletedText = this.document.deleteRange(this.start, this.end)
        this.executed = true
    }

    method undo(): void {
        // Re-insert the deleted text
        this.document.insertAt(this.start, this.deletedText)
        this.executed = false
    }

    method getDescription(): string {
        preview = this.deletedText.substring(0, min(20, this.deletedText.length))
        if this.deletedText.length > 20 {
            preview = preview + "..."
        }
        return "Delete: \"" + preview + "\""
    }
}

// Concrete Command: Copy (doesn't modify document, so no undo needed)
class CopyCommand implements Command {
    private document: TextDocument

    constructor(document: TextDocument) {
        this.document = document
    }

    method execute(): void {
        selectedText = this.document.getSelectedText()
        this.document.setClipboard(selectedText)
    }

    method undo(): void {
        // Copy doesn't modify document, nothing to undo
    }

    method redo(): void {
        this.execute()
    }

    method getDescription(): string {
        return "Copy"
    }
}

// Concrete Command: Paste
class PasteCommand extends UndoableCommand {
    private document: TextDocument
    private position: int
    private pastedText: string = ""

    constructor(document: TextDocument, position: int) {
        this.document = document
        this.position = position
    }

    method execute(): void {
        this.pastedText = this.document.getClipboard()
        this.document.insertAt(this.position, this.pastedText)
        this.executed = true
    }

    method undo(): void {
        this.document.deleteRange(this.position, this.position + this.pastedText.length)
        this.executed = false
    }

    method getDescription(): string {
        return "Paste"
    }
}

// Concrete Command: Format Text (Bold, Italic, etc.)
class FormatTextCommand extends UndoableCommand {
    private document: TextDocument
    private range: Range
    private style: FormatStyle
    private previousStyle: FormatStyle = null

    constructor(document: TextDocument, range: Range, style: FormatStyle) {
        this.document = document
        this.range = range
        this.style = style
    }

    method execute(): void {
        // Store previous formatting for undo
        this.previousStyle = this.document.getFormatting(this.range)
        this.document.applyFormatting(this.range, this.style)
        this.executed = true
    }

    method undo(): void {
        if this.previousStyle is not null {
            this.document.applyFormatting(this.range, this.previousStyle)
        } else {
            this.document.removeFormatting(this.range)
        }
        this.executed = false
    }

    method getDescription(): string {
        return "Format: " + this.style.name
    }
}
```

### Pseudocode: Command History (Invoker)

```
class CommandHistory {
    private history: list<Command> = []
    private currentIndex: int = -1
    private maxHistorySize: int = 100

    method execute(command: Command): void {
        // Clear any commands after current position (invalidated by new action)
        while this.history.size() > this.currentIndex + 1 {
            this.history.removeLast()
        }

        // Execute the command
        command.execute()

        // Add to history
        this.history.add(command)
        this.currentIndex = this.history.size() - 1

        // Trim history if too large
        if this.history.size() > this.maxHistorySize {
            this.history.removeFirst()
            this.currentIndex = this.currentIndex - 1
        }
    }

    method undo(): boolean {
        if not this.canUndo() {
            return false
        }

        command = this.history.get(this.currentIndex)
        command.undo()
        this.currentIndex = this.currentIndex - 1
        return true
    }

    method redo(): boolean {
        if not this.canRedo() {
            return false
        }

        this.currentIndex = this.currentIndex + 1
        command = this.history.get(this.currentIndex)
        command.redo()
        return true
    }

    method canUndo(): boolean {
        return this.currentIndex >= 0
    }

    method canRedo(): boolean {
        return this.currentIndex < this.history.size() - 1
    }

    method getUndoDescription(): string {
        if this.canUndo() {
            return this.history.get(this.currentIndex).getDescription()
        }
        return null
    }

    method getRedoDescription(): string {
        if this.canRedo() {
            return this.history.get(this.currentIndex + 1).getDescription()
        }
        return null
    }

    method clear(): void {
        this.history.clear()
        this.currentIndex = -1
    }
}
```

## Example

A complete example implementing a drawing application with command pattern:

```
// Receiver: Canvas that holds shapes
class DrawingCanvas {
    private shapes: list<Shape> = []
    private selectedShapes: list<Shape> = []

    method addShape(shape: Shape): void {
        this.shapes.add(shape)
    }

    method removeShape(shape: Shape): void {
        this.shapes.remove(shape)
        this.selectedShapes.remove(shape)
    }

    method getShapeAt(x: int, y: int): Shape {
        // Return topmost shape at coordinates (reverse iteration)
        for i = this.shapes.size() - 1 downto 0 {
            if this.shapes.get(i).containsPoint(x, y) {
                return this.shapes.get(i)
            }
        }
        return null
    }

    method getAllShapes(): list<Shape> {
        return this.shapes.copy()
    }

    method setSelection(shapes: list<Shape>): void {
        this.selectedShapes = shapes
    }

    method getSelection(): list<Shape> {
        return this.selectedShapes.copy()
    }
}

// Shape base class
abstract class Shape {
    public id: string
    public x: int
    public y: int
    public width: int
    public height: int
    public color: Color
    public strokeWidth: int

    method containsPoint(px: int, py: int): boolean {
        return px >= this.x and px <= this.x + this.width and
               py >= this.y and py <= this.y + this.height
    }

    method move(dx: int, dy: int): void {
        this.x = this.x + dx
        this.y = this.y + dy
    }

    method clone(): Shape {
        // Deep copy
    }
}

class Rectangle extends Shape {
    public cornerRadius: int = 0
}

class Ellipse extends Shape {
}

class TextBox extends Shape {
    public text: string
    public font: Font
}

// Command: Add Shape
class AddShapeCommand extends UndoableCommand {
    private canvas: DrawingCanvas
    private shape: Shape

    constructor(canvas: DrawingCanvas, shape: Shape) {
        this.canvas = canvas
        this.shape = shape
    }

    method execute(): void {
        this.canvas.addShape(this.shape)
        this.executed = true
    }

    method undo(): void {
        this.canvas.removeShape(this.shape)
        this.executed = false
    }

    method getDescription(): string {
        return "Add " + this.shape.getClass().name
    }
}

// Command: Delete Shapes
class DeleteShapesCommand extends UndoableCommand {
    private canvas: DrawingCanvas
    private shapes: list<Shape>
    private originalIndices: list<int>

    constructor(canvas: DrawingCanvas, shapes: list<Shape>) {
        this.canvas = canvas
        this.shapes = shapes.copy()
        this.originalIndices = []
    }

    method execute(): void {
        // Store original positions for proper undo
        allShapes = this.canvas.getAllShapes()
        for shape in this.shapes {
            index = allShapes.indexOf(shape)
            this.originalIndices.add(index)
        }

        // Remove shapes (in reverse order to preserve indices)
        for shape in this.shapes.reversed() {
            this.canvas.removeShape(shape)
        }
        this.executed = true
    }

    method undo(): void {
        // Re-add shapes at original positions
        for i = 0 to this.shapes.size() - 1 {
            // Note: This is simplified; real implementation would restore exact z-order
            this.canvas.addShape(this.shapes.get(i))
        }
        this.executed = false
    }

    method getDescription(): string {
        return "Delete " + this.shapes.size() + " shape(s)"
    }
}

// Command: Move Shapes
class MoveShapesCommand extends UndoableCommand {
    private canvas: DrawingCanvas
    private shapes: list<Shape>
    private dx: int
    private dy: int

    constructor(canvas: DrawingCanvas, shapes: list<Shape>, dx: int, dy: int) {
        this.canvas = canvas
        this.shapes = shapes.copy()
        this.dx = dx
        this.dy = dy
    }

    method execute(): void {
        for shape in this.shapes {
            shape.move(this.dx, this.dy)
        }
        this.executed = true
    }

    method undo(): void {
        for shape in this.shapes {
            shape.move(-this.dx, -this.dy)
        }
        this.executed = false
    }

    method getDescription(): string {
        return "Move " + this.shapes.size() + " shape(s)"
    }

    // Allow combining consecutive move commands
    method canMergeWith(other: Command): boolean {
        if other is not MoveShapesCommand {
            return false
        }
        otherMove = other as MoveShapesCommand
        return this.shapes.equals(otherMove.shapes)
    }

    method mergeWith(other: MoveShapesCommand): MoveShapesCommand {
        return new MoveShapesCommand(
            this.canvas,
            this.shapes,
            this.dx + other.dx,
            this.dy + other.dy
        )
    }
}

// Command: Change Color
class ChangeColorCommand extends UndoableCommand {
    private shapes: list<Shape>
    private newColor: Color
    private previousColors: map<string, Color>

    constructor(shapes: list<Shape>, newColor: Color) {
        this.shapes = shapes.copy()
        this.newColor = newColor
        this.previousColors = {}
    }

    method execute(): void {
        for shape in this.shapes {
            this.previousColors.set(shape.id, shape.color)
            shape.color = this.newColor
        }
        this.executed = true
    }

    method undo(): void {
        for shape in this.shapes {
            shape.color = this.previousColors.get(shape.id)
        }
        this.executed = false
    }

    method getDescription(): string {
        return "Change color to " + this.newColor.name
    }
}

// Command: Group Shapes (Composite Command / Macro)
class GroupShapesCommand extends UndoableCommand {
    private canvas: DrawingCanvas
    private shapes: list<Shape>
    private group: ShapeGroup

    constructor(canvas: DrawingCanvas, shapes: list<Shape>) {
        this.canvas = canvas
        this.shapes = shapes.copy()
    }

    method execute(): void {
        // Create group
        this.group = new ShapeGroup()
        for shape in this.shapes {
            this.canvas.removeShape(shape)
            this.group.addChild(shape)
        }
        this.canvas.addShape(this.group)
        this.executed = true
    }

    method undo(): void {
        // Ungroup
        this.canvas.removeShape(this.group)
        for shape in this.shapes {
            this.canvas.addShape(shape)
        }
        this.executed = false
    }

    method getDescription(): string {
        return "Group " + this.shapes.size() + " shapes"
    }
}

// Macro Command: Duplicate Shapes (composed of multiple commands)
class MacroCommand implements Command {
    private commands: list<Command> = []
    private description: string

    constructor(description: string) {
        this.description = description
    }

    method add(command: Command): void {
        this.commands.add(command)
    }

    method execute(): void {
        for command in this.commands {
            command.execute()
        }
    }

    method undo(): void {
        // Undo in reverse order
        for command in this.commands.reversed() {
            command.undo()
        }
    }

    method redo(): void {
        this.execute()
    }

    method getDescription(): string {
        return this.description
    }
}

// Factory for creating duplicate command
class DuplicateCommandFactory {
    static method create(canvas: DrawingCanvas, shapes: list<Shape>): Command {
        macro = new MacroCommand("Duplicate " + shapes.size() + " shape(s)")

        for shape in shapes {
            clone = shape.clone()
            clone.id = generateUUID()
            clone.move(20, 20)  // Offset the duplicate

            macro.add(new AddShapeCommand(canvas, clone))
        }

        return macro
    }
}

// Drawing Application (Client + Invoker)
class DrawingApplication {
    private canvas: DrawingCanvas
    private history: CommandHistory
    private clipboard: list<Shape> = []

    constructor() {
        this.canvas = new DrawingCanvas()
        this.history = new CommandHistory()
    }

    // User actions that create and execute commands

    method addRectangle(x: int, y: int, width: int, height: int): void {
        rect = new Rectangle()
        rect.id = generateUUID()
        rect.x = x
        rect.y = y
        rect.width = width
        rect.height = height
        rect.color = Color.BLUE

        command = new AddShapeCommand(this.canvas, rect)
        this.history.execute(command)
    }

    method deleteSelected(): void {
        selected = this.canvas.getSelection()
        if selected.isEmpty() {
            return
        }

        command = new DeleteShapesCommand(this.canvas, selected)
        this.history.execute(command)
    }

    method moveSelected(dx: int, dy: int): void {
        selected = this.canvas.getSelection()
        if selected.isEmpty() {
            return
        }

        command = new MoveShapesCommand(this.canvas, selected, dx, dy)
        this.history.execute(command)
    }

    method changeColor(color: Color): void {
        selected = this.canvas.getSelection()
        if selected.isEmpty() {
            return
        }

        command = new ChangeColorCommand(selected, color)
        this.history.execute(command)
    }

    method duplicateSelected(): void {
        selected = this.canvas.getSelection()
        if selected.isEmpty() {
            return
        }

        command = DuplicateCommandFactory.create(this.canvas, selected)
        this.history.execute(command)
    }

    method groupSelected(): void {
        selected = this.canvas.getSelection()
        if selected.size() < 2 {
            return
        }

        command = new GroupShapesCommand(this.canvas, selected)
        this.history.execute(command)
    }

    method undo(): void {
        if this.history.canUndo() {
            this.history.undo()
        }
    }

    method redo(): void {
        if this.history.canRedo() {
            this.history.redo()
        }
    }

    method getUndoMenuText(): string {
        desc = this.history.getUndoDescription()
        if desc is not null {
            return "Undo " + desc
        }
        return "Undo"
    }

    method getRedoMenuText(): string {
        desc = this.history.getRedoDescription()
        if desc is not null {
            return "Redo " + desc
        }
        return "Redo"
    }
}

// Usage
function main() {
    app = new DrawingApplication()

    // User draws some shapes
    app.addRectangle(100, 100, 50, 50)
    app.addRectangle(200, 200, 75, 50)

    // Select and move
    app.canvas.setSelection([...])
    app.moveSelected(10, 10)

    // Change color
    app.changeColor(Color.RED)

    // Undo the color change
    app.undo()  // Shapes are blue again

    // Redo
    app.redo()  // Shapes are red again

    // Undo multiple times
    app.undo()  // Color undone
    app.undo()  // Move undone
    app.undo()  // Second rectangle undone
    app.undo()  // First rectangle undone
    // Canvas is now empty

    print(app.getUndoMenuText())  // "Undo" (nothing to undo)
    print(app.getRedoMenuText())  // "Redo Add Rectangle"
}
```

## Known Uses

- **Text Editors**: Microsoft Word, VSCode, and virtually all text editors use Command pattern for edit operations and undo/redo.

- **GUI Frameworks**: Qt's QUndoCommand, WPF's ICommand interface, and Swing's Action interface are all Command pattern implementations.

- **Transaction Systems**: Database transactions encapsulate operations that can be committed or rolled back.

- **Game Development**: Most games use command pattern for player actions, enabling replay systems, networked gameplay, and undo in level editors.

- **Version Control**: Git commits are essentially commands that can be applied (checkout) or reversed (revert).

- **Job Queues**: Systems like Sidekiq, Celery, and RabbitMQ use serialized command objects for background job processing.

- **Redux**: The popular state management library models all state changes as action objects (commands) dispatched to reducers.

- **Event Sourcing**: Architectural pattern where all changes are stored as a sequence of commands/events.

- **Macro Recording**: Applications like Photoshop and Excel record user actions as commands for playback.

## Related Patterns

- **Composite**: MacroCommand (composite commands) use the Composite pattern. Commands can contain other commands.

- **Memento**: Commands often use Memento to store state for undo operations. The command stores a memento of the receiver before executing.

- **Prototype**: When commands need to be copied (e.g., for history or logging), Prototype pattern helps.

- **Chain of Responsibility**: Commands can be processed through a chain of handlers, combining both patterns.

- **Strategy**: Both encapsulate algorithms/behavior in objects. Command is about "what to do" (request encapsulation), while Strategy is about "how to do it" (algorithm selection).

- **Visitor**: Visitor defines operations on object structures; Command encapsulates single operations for execution control.

## When NOT to Use

1. **Simple, non-undoable operations**: If you don't need undo/redo, queuing, or logging, direct method calls are simpler.

```
// Over-engineering: Using Command for simple one-shot operations
class PrintMessageCommand {
    constructor(message: string)
    method execute(): void {
        print(message)
    }
}

// Just do this instead:
print(message)
```

2. **When commands have no meaningful undo**: Some operations (sending an email, API calls with side effects) can't truly be undone. Command pattern's undo capability doesn't help here.

3. **Performance-critical tight loops**: The indirection and object creation overhead of Command pattern can impact performance in hot paths.

4. **When receivers frequently change**: If the receiver's interface changes often, you'll need to update all related commands. Direct coupling might be easier to maintain.

5. **Stateless operations**: If operations don't need to store state and don't benefit from being objects, Command adds unnecessary complexity.

6. **Simple CRUD operations**: Basic create/read/update/delete operations often don't need the full Command infrastructure unless you specifically need audit logging or undo.

7. **When command lifecycle is trivial**: If commands are created, executed once, and immediately discarded, you're not using the pattern's strengths.

```
// Anti-pattern: Creating commands just to execute them once
command = new SaveFileCommand(document, filename)
command.execute()
// Command is never used again - just call document.save(filename)
```

The Command pattern is powerful when you need to treat operations as first-class objects—for undo/redo, queuing, logging, or transactions. If you only need to execute code, simpler approaches will serve you better.

---

*Based on concepts from "Design Patterns: Elements of Reusable Object-Oriented Software" by Gamma, Helm, Johnson, and Vlissides (Gang of Four), 1994.*
