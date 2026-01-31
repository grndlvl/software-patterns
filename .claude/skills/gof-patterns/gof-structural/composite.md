# Composite

## Intent

Compose objects into tree structures to represent part-whole hierarchies. Composite lets clients treat individual objects and compositions of objects uniformly. This pattern allows you to build complex nested structures while maintaining a simple interface for interacting with both simple and compound elements.

## Also Known As

- Object Tree
- Recursive Composition

## Motivation

Graphics applications like drawing editors and schematic capture systems allow users to build complex diagrams out of simple components. Users can group components to form larger components, which in turn can be grouped to form still larger components. A simple implementation could define classes for graphical primitives such as Text and Lines plus other classes that act as containers for these primitives.

The problem with this approach is that code that uses these classes must treat primitive and container objects differently, even when the user treats them identically. Having to distinguish these objects makes the application more complex. The Composite pattern describes how to use recursive composition so that clients do not have to make this distinction.

Consider a file system. A file system consists of directories and files. A directory can contain files and other directories. When you want to calculate the size of a directory, you sum the sizes of all files and recursively sum the sizes of all subdirectories. The operation is the same regardless of whether you are dealing with a file or a directory - you ask for its size. The Composite pattern captures this uniformity.

The key to the Composite pattern is an abstract class (Component) that represents both primitives and their containers. The Component declares operations like `getSize()` that are common to both. It also declares operations for accessing and managing children, although leaves (primitives) may not use these. Leaves define behavior for primitives, while Composites define behavior for components having children and store child components.

## Applicability

Use the Composite pattern when:

- You want to represent part-whole hierarchies of objects.

- You want clients to be able to ignore the difference between compositions of objects and individual objects. Clients will treat all objects in the composite structure uniformly.

- You have a tree structure where branches and leaves share common operations.

- You want to simplify client code by allowing it to work with complex tree structures using simple method calls.

- You need to perform operations across an entire hierarchy, such as calculating totals, searching, or rendering.

- You want to add new component types without changing existing code (Open/Closed Principle).

## Structure

```
                    ┌─────────────────────────┐
                    │    <<interface>>        │
                    │      Component          │
                    ├─────────────────────────┤
                    │ + operation()           │
                    │ + add(c: Component)     │
                    │ + remove(c: Component)  │
                    │ + getChild(i): Component│
                    └─────────────────────────┘
                                △
                                │
                ┌───────────────┴───────────────┐
                │                               │
    ┌───────────────────┐           ┌───────────────────────┐
    │       Leaf        │           │      Composite        │
    ├───────────────────┤           ├───────────────────────┤
    │ + operation()     │           │ - children: List      │
    │   // Leaf-specific│           ├───────────────────────┤
    │   // behavior     │           │ + operation()         │
    └───────────────────┘           │   for child in children│
                                    │     child.operation() │
                                    │ + add(c: Component)   │
                                    │ + remove(c: Component)│
                                    │ + getChild(i)         │
                                    └───────────────────────┘
                                              │
                                              │ contains
                                              ▼
                                    ┌───────────────────┐
                                    │    Component *    │
                                    └───────────────────┘
```

### Tree Structure Example

```
                        ┌─────────────┐
                        │  Composite  │
                        │   (Root)    │
                        └─────────────┘
                              │
            ┌─────────────────┼─────────────────┐
            │                 │                 │
      ┌───────────┐    ┌───────────┐    ┌───────────┐
      │   Leaf    │    │ Composite │    │   Leaf    │
      └───────────┘    └───────────┘    └───────────┘
                             │
                    ┌────────┴────────┐
                    │                 │
              ┌───────────┐    ┌───────────┐
              │   Leaf    │    │   Leaf    │
              └───────────┘    └───────────┘
```

## Participants

- **Component**: Declares the interface for objects in the composition. Implements default behavior for the interface common to all classes, as appropriate. Declares an interface for accessing and managing its child components. Optionally defines an interface for accessing a component's parent in the recursive structure.

- **Leaf**: Represents leaf objects in the composition. A leaf has no children. Defines behavior for primitive objects in the composition.

- **Composite**: Defines behavior for components having children. Stores child components. Implements child-related operations in the Component interface.

- **Client**: Manipulates objects in the composition through the Component interface. The client is unaware of whether it is working with a Leaf or a Composite.

## Collaborations

1. Clients use the Component class interface to interact with objects in the composite structure.

2. If the recipient is a Leaf, the request is handled directly.

3. If the recipient is a Composite, it usually forwards requests to its child components, possibly performing additional operations before and/or after forwarding.

4. The forwarding typically involves iterating over all children and invoking the same operation on each, aggregating results as appropriate.

## Consequences

### Benefits

- **Uniform treatment**: Clients can treat composite structures and individual objects uniformly. Client code that uses the Component interface can work with either leaves or composites without knowing which.

- **Simplified client code**: Clients do not need to know whether they are dealing with a leaf or a composite. They can treat all components the same way.

- **Easy to add new component types**: New Leaf or Composite classes can be added without changing existing code. The pattern is open for extension.

- **Natural tree representation**: The pattern provides a natural way to represent tree-structured data, such as file systems, UI component hierarchies, or organizational structures.

- **Recursive operations**: Operations can be applied recursively across the entire tree structure with simple method calls.

- **Flexible structure**: The structure can be composed at runtime, allowing dynamic construction of complex hierarchies.

### Liabilities

- **Overly general design**: The pattern can make the design overly general. It becomes harder to restrict the types of components in a composite. You cannot rely on the type system to enforce constraints.

- **Type safety challenges**: Child management operations (add, remove, getChild) make sense for Composites but not for Leaves. Different approaches (with tradeoffs) handle this.

- **Difficult to enforce constraints**: It is harder to enforce rules about what types of children a composite can have. Such constraints must be checked at runtime.

- **Inefficient for large structures**: Operations that traverse the entire tree can be expensive for very large structures. Consider caching or lazy evaluation.

- **Parent references complicate modifications**: If components keep references to their parents, adding or removing children must maintain this bidirectional relationship.

## Implementation

### Design Decisions

1. **Explicit parent references**: Should Component maintain a reference to its parent? This simplifies traversal and moving up the tree but requires updating parent references when children are added or removed.

2. **Component interface completeness**: Should Component declare all child management operations? Two main approaches:
   - **Transparency**: Declare add/remove/getChild in Component, defaulting to no-op or error in Leaf. Maximizes transparency but sacrifices type safety.
   - **Safety**: Only declare in Composite. Maximizes safety but clients may need to check types.

3. **Child ordering**: Should children be ordered? Lists maintain insertion order; Sets do not but prevent duplicates.

4. **Caching**: For expensive traversal operations, consider caching computed values in composites and invalidating when children change.

5. **Who deletes children**: In languages with manual memory management, decide whether Composite owns its children (deletes them when deleted) or just references them.

### Transparent Component Interface

```
// Component interface - transparent design with all operations
interface FileSystemNode
    method getName(): String
    method getSize(): Integer
    method getPath(): String

    // Child management - in Component for transparency
    method add(child: FileSystemNode)
    method remove(child: FileSystemNode)
    method getChildren(): List<FileSystemNode>

    // Optional: parent reference
    method getParent(): FileSystemNode
    method setParent(parent: FileSystemNode)

    // Tree operations
    method find(name: String): FileSystemNode
    method list(indent: Integer)
end interface

// Leaf - represents a file
class File implements FileSystemNode
    private name: String
    private size: Integer
    private parent: FileSystemNode

    constructor(name: String, size: Integer)
        this.name = name
        this.size = size
    end constructor

    method getName(): String
        return name
    end method

    method getSize(): Integer
        return size
    end method

    method getPath(): String
        if parent == null
            return "/" + name
        end if
        return parent.getPath() + "/" + name
    end method

    // Leaf doesn't support children - these could throw or no-op
    method add(child: FileSystemNode)
        throw new UnsupportedOperationException("Files cannot contain children")
    end method

    method remove(child: FileSystemNode)
        throw new UnsupportedOperationException("Files cannot contain children")
    end method

    method getChildren(): List<FileSystemNode>
        return emptyList()  // Or could throw
    end method

    method getParent(): FileSystemNode
        return parent
    end method

    method setParent(parent: FileSystemNode)
        this.parent = parent
    end method

    method find(name: String): FileSystemNode
        if this.name == name
            return this
        end if
        return null
    end method

    method list(indent: Integer)
        print " ".repeat(indent) + name + " (" + size + " bytes)"
    end method
end class

// Composite - represents a directory
class Directory implements FileSystemNode
    private name: String
    private children: List<FileSystemNode>
    private parent: FileSystemNode
    private cachedSize: Integer
    private sizeValid: Boolean

    constructor(name: String)
        this.name = name
        this.children = new ArrayList()
        this.sizeValid = false
    end constructor

    method getName(): String
        return name
    end method

    method getSize(): Integer
        // Use cached value if valid
        if sizeValid
            return cachedSize
        end if

        // Calculate recursively
        total = 0
        for each child in children
            total = total + child.getSize()
        end for

        cachedSize = total
        sizeValid = true
        return total
    end method

    method getPath(): String
        if parent == null
            return "/" + name
        end if
        return parent.getPath() + "/" + name
    end method

    method add(child: FileSystemNode)
        children.add(child)
        child.setParent(this)
        invalidateSize()
    end method

    method remove(child: FileSystemNode)
        children.remove(child)
        child.setParent(null)
        invalidateSize()
    end method

    method getChildren(): List<FileSystemNode>
        return Collections.unmodifiableList(children)
    end method

    method getParent(): FileSystemNode
        return parent
    end method

    method setParent(parent: FileSystemNode)
        this.parent = parent
    end method

    method find(name: String): FileSystemNode
        if this.name == name
            return this
        end if

        // Search children recursively
        for each child in children
            result = child.find(name)
            if result != null
                return result
            end if
        end for

        return null
    end method

    method list(indent: Integer)
        print " ".repeat(indent) + "[" + name + "]"
        for each child in children
            child.list(indent + 2)
        end for
    end method

    private method invalidateSize()
        sizeValid = false
        // Propagate invalidation up the tree
        if parent != null and parent instanceof Directory
            (parent as Directory).invalidateSize()
        end if
    end method
end class
```

### Safe Component Interface

```
// Safer design - child operations only on Composite
interface Component
    method operation(): Result
    method isComposite(): Boolean
end interface

interface CompositeComponent extends Component
    method add(child: Component)
    method remove(child: Component)
    method getChildren(): List<Component>
end interface

class Leaf implements Component
    method operation(): Result
        // Leaf-specific behavior
    end method

    method isComposite(): Boolean
        return false
    end method
end class

class Composite implements CompositeComponent
    private children: List<Component>

    method operation(): Result
        // Aggregate children's results
        results = new List()
        for each child in children
            results.add(child.operation())
        end for
        return aggregateResults(results)
    end method

    method isComposite(): Boolean
        return true
    end method

    method add(child: Component)
        children.add(child)
    end method

    method remove(child: Component)
        children.remove(child)
    end method

    method getChildren(): List<Component>
        return children
    end method
end class

// Client must check type before child operations
method addToComposite(component: Component, child: Component)
    if component.isComposite()
        (component as CompositeComponent).add(child)
    else
        throw new IllegalArgumentException("Cannot add to leaf")
    end if
end method
```

## Example

### UI Component Hierarchy

A real-world example of a UI framework where components can contain other components, forming complex layouts.

```
// Component interface for UI elements
interface UIComponent
    // Rendering
    method render(context: RenderContext)
    method getBounds(): Rectangle
    method setPosition(x: Integer, y: Integer)
    method setSize(width: Integer, height: Integer)

    // Child management (transparent design)
    method addChild(child: UIComponent)
    method removeChild(child: UIComponent)
    method getChildren(): List<UIComponent>

    // Events
    method handleEvent(event: UIEvent): Boolean
    method findComponentAt(x: Integer, y: Integer): UIComponent

    // Properties
    method setVisible(visible: Boolean)
    method isVisible(): Boolean
    method setEnabled(enabled: Boolean)
    method isEnabled(): Boolean

    // Layout
    method layout()
    method getPreferredSize(): Dimension
end interface

// Abstract base with common functionality
abstract class AbstractUIComponent implements UIComponent
    protected x: Integer
    protected y: Integer
    protected width: Integer
    protected height: Integer
    protected visible: Boolean
    protected enabled: Boolean
    protected parent: UIComponent

    constructor()
        this.visible = true
        this.enabled = true
    end constructor

    method getBounds(): Rectangle
        return new Rectangle(x, y, width, height)
    end method

    method setPosition(x: Integer, y: Integer)
        this.x = x
        this.y = y
    end method

    method setSize(width: Integer, height: Integer)
        this.width = width
        this.height = height
    end method

    method setVisible(visible: Boolean)
        this.visible = visible
    end method

    method isVisible(): Boolean
        return visible
    end method

    method setEnabled(enabled: Boolean)
        this.enabled = enabled
    end method

    method isEnabled(): Boolean
        return enabled
    end method

    // Default implementations for leaf components
    method addChild(child: UIComponent)
        throw new UnsupportedOperationException("Cannot add children to this component")
    end method

    method removeChild(child: UIComponent)
        throw new UnsupportedOperationException("Cannot remove children from this component")
    end method

    method getChildren(): List<UIComponent>
        return emptyList()
    end method

    method layout()
        // Default: no layout needed for leaves
    end method
end class

// Leaf: Button
class Button extends AbstractUIComponent
    private text: String
    private onClick: Function
    private style: ButtonStyle

    constructor(text: String)
        super()
        this.text = text
        this.style = ButtonStyle.DEFAULT
    end constructor

    method setText(text: String)
        this.text = text
    end method

    method setOnClick(handler: Function)
        this.onClick = handler
    end method

    method setStyle(style: ButtonStyle)
        this.style = style
    end method

    method render(context: RenderContext)
        if not visible
            return
        end if

        // Draw button background
        context.setFillColor(style.backgroundColor)
        context.fillRoundedRect(x, y, width, height, style.borderRadius)

        // Draw border
        context.setStrokeColor(style.borderColor)
        context.strokeRoundedRect(x, y, width, height, style.borderRadius)

        // Draw text centered
        context.setFont(style.font)
        context.setTextColor(style.textColor)
        textWidth = context.measureText(text)
        textX = x + (width - textWidth) / 2
        textY = y + (height + style.font.size) / 2
        context.drawText(text, textX, textY)
    end method

    method handleEvent(event: UIEvent): Boolean
        if not enabled or not visible
            return false
        end if

        if event.type == EventType.CLICK
            if getBounds().contains(event.x, event.y)
                if onClick != null
                    onClick()
                end if
                return true
            end if
        end if

        return false
    end method

    method findComponentAt(x: Integer, y: Integer): UIComponent
        if visible and getBounds().contains(x, y)
            return this
        end if
        return null
    end method

    method getPreferredSize(): Dimension
        // Calculate based on text and padding
        textWidth = measureText(text, style.font)
        return new Dimension(
            textWidth + style.paddingLeft + style.paddingRight,
            style.font.size + style.paddingTop + style.paddingBottom
        )
    end method
end class

// Leaf: TextInput
class TextInput extends AbstractUIComponent
    private value: String
    private placeholder: String
    private onChange: Function

    constructor(placeholder: String)
        super()
        this.placeholder = placeholder
        this.value = ""
    end constructor

    method getValue(): String
        return value
    end method

    method setValue(value: String)
        this.value = value
        if onChange != null
            onChange(value)
        end if
    end method

    method render(context: RenderContext)
        if not visible
            return
        end if

        // Draw input box
        context.setFillColor(Color.WHITE)
        context.fillRect(x, y, width, height)
        context.setStrokeColor(Color.GRAY)
        context.strokeRect(x, y, width, height)

        // Draw text or placeholder
        displayText = value.isEmpty() ? placeholder : value
        textColor = value.isEmpty() ? Color.LIGHT_GRAY : Color.BLACK
        context.setTextColor(textColor)
        context.drawText(displayText, x + 5, y + height / 2)
    end method

    method handleEvent(event: UIEvent): Boolean
        if not enabled or not visible
            return false
        end if

        if event.type == EventType.KEY_PRESS
            if getBounds().contains(event.x, event.y)
                value = value + event.key
                if onChange != null
                    onChange(value)
                end if
                return true
            end if
        end if

        return false
    end method

    method findComponentAt(x: Integer, y: Integer): UIComponent
        if visible and getBounds().contains(x, y)
            return this
        end if
        return null
    end method

    method getPreferredSize(): Dimension
        return new Dimension(200, 30)  // Default size for text input
    end method
end class

// Leaf: Label
class Label extends AbstractUIComponent
    private text: String
    private font: Font
    private color: Color

    constructor(text: String)
        super()
        this.text = text
        this.font = Font.DEFAULT
        this.color = Color.BLACK
    end constructor

    method render(context: RenderContext)
        if not visible
            return
        end if

        context.setFont(font)
        context.setTextColor(color)
        context.drawText(text, x, y + font.size)
    end method

    method handleEvent(event: UIEvent): Boolean
        return false  // Labels don't handle events
    end method

    method findComponentAt(x: Integer, y: Integer): UIComponent
        if visible and getBounds().contains(x, y)
            return this
        end if
        return null
    end method

    method getPreferredSize(): Dimension
        textWidth = measureText(text, font)
        return new Dimension(textWidth, font.size)
    end method
end class

// Composite: Panel (container for other components)
class Panel extends AbstractUIComponent
    protected children: List<UIComponent>
    protected layout: LayoutManager
    protected backgroundColor: Color
    protected padding: Insets

    constructor()
        super()
        this.children = new ArrayList()
        this.layout = new FlowLayout()
        this.backgroundColor = Color.TRANSPARENT
        this.padding = new Insets(0, 0, 0, 0)
    end constructor

    method setLayout(layout: LayoutManager)
        this.layout = layout
        layout()
    end method

    method setBackgroundColor(color: Color)
        this.backgroundColor = color
    end method

    method setPadding(padding: Insets)
        this.padding = padding
        layout()
    end method

    // Child management
    method addChild(child: UIComponent)
        children.add(child)
        child.parent = this
        layout()
    end method

    method removeChild(child: UIComponent)
        children.remove(child)
        child.parent = null
        layout()
    end method

    method getChildren(): List<UIComponent>
        return Collections.unmodifiableList(children)
    end method

    // Rendering - composites render themselves then children
    method render(context: RenderContext)
        if not visible
            return
        end if

        // Draw background
        if backgroundColor != Color.TRANSPARENT
            context.setFillColor(backgroundColor)
            context.fillRect(x, y, width, height)
        end if

        // Create clipping region to prevent children from drawing outside
        context.pushClip(x, y, width, height)

        // Render all children
        for each child in children
            child.render(context)
        end for

        context.popClip()
    end method

    // Events propagate to children
    method handleEvent(event: UIEvent): Boolean
        if not enabled or not visible
            return false
        end if

        // Try children first (front to back for clicks)
        for i = children.size() - 1 downto 0
            child = children.get(i)
            if child.handleEvent(event)
                return true
            end if
        end for

        return false
    end method

    method findComponentAt(x: Integer, y: Integer): UIComponent
        if not visible or not getBounds().contains(x, y)
            return null
        end if

        // Check children (front to back)
        for i = children.size() - 1 downto 0
            child = children.get(i)
            found = child.findComponentAt(x, y)
            if found != null
                return found
            end if
        end for

        return this
    end method

    method layout()
        if layout != null
            layout.doLayout(this, children, padding)
        end if
    end method

    method getPreferredSize(): Dimension
        if layout != null
            return layout.preferredSize(children, padding)
        end if
        return new Dimension(width, height)
    end method
end class

// Composite: Form (specialized container)
class Form extends Panel
    private onSubmit: Function
    private submitButton: Button

    constructor()
        super()
        this.setLayout(new VerticalLayout(10))

        // Auto-add submit button
        submitButton = new Button("Submit")
        submitButton.setOnClick(() -> submit())
    end constructor

    method addField(label: String, input: UIComponent)
        row = new Panel()
        row.setLayout(new HorizontalLayout(10))
        row.addChild(new Label(label))
        row.addChild(input)
        this.addChild(row)
    end method

    method setOnSubmit(handler: Function)
        this.onSubmit = handler
    end method

    method showSubmitButton()
        if not children.contains(submitButton)
            this.addChild(submitButton)
        end if
    end method

    method submit()
        if onSubmit != null
            formData = collectFormData()
            onSubmit(formData)
        end if
    end method

    private method collectFormData(): Map
        data = new Map()
        for each child in children
            if child instanceof Panel
                for each grandchild in child.getChildren()
                    if grandchild instanceof TextInput
                        // Use label text as key
                        data.set(getLabelForInput(child), grandchild.getValue())
                    end if
                end for
            end if
        end for
        return data
    end method
end class

// Composite: ScrollablePanel
class ScrollablePanel extends Panel
    private scrollX: Integer
    private scrollY: Integer
    private contentWidth: Integer
    private contentHeight: Integer

    constructor()
        super()
        this.scrollX = 0
        this.scrollY = 0
    end constructor

    method render(context: RenderContext)
        if not visible
            return
        end if

        // Draw background
        if backgroundColor != Color.TRANSPARENT
            context.setFillColor(backgroundColor)
            context.fillRect(x, y, width, height)
        end if

        // Clip to visible area and translate by scroll offset
        context.pushClip(x, y, width, height)
        context.translate(-scrollX, -scrollY)

        // Render all children
        for each child in children
            child.render(context)
        end for

        context.translate(scrollX, scrollY)
        context.popClip()

        // Draw scrollbars if needed
        drawScrollbars(context)
    end method

    method handleEvent(event: UIEvent): Boolean
        if event.type == EventType.SCROLL
            if getBounds().contains(event.x, event.y)
                scrollY = Math.clamp(scrollY + event.scrollDelta, 0, maxScrollY())
                return true
            end if
        end if

        // Adjust event coordinates for scroll offset
        adjustedEvent = event.translate(scrollX, scrollY)
        return super.handleEvent(adjustedEvent)
    end method

    method scrollTo(x: Integer, y: Integer)
        this.scrollX = Math.clamp(x, 0, maxScrollX())
        this.scrollY = Math.clamp(y, 0, maxScrollY())
    end method

    private method maxScrollX(): Integer
        return Math.max(0, contentWidth - width)
    end method

    private method maxScrollY(): Integer
        return Math.max(0, contentHeight - height)
    end method
end class

// Usage example - building a complex UI
method createLoginForm(): UIComponent
    // Create the form container
    form = new Form()
    form.setPosition(50, 50)
    form.setSize(300, 200)
    form.setBackgroundColor(Color.LIGHT_GRAY)
    form.setPadding(new Insets(20, 20, 20, 20))

    // Add fields
    usernameInput = new TextInput("Enter username")
    form.addField("Username:", usernameInput)

    passwordInput = new TextInput("Enter password")
    form.addField("Password:", passwordInput)

    // Add buttons in a horizontal panel
    buttonPanel = new Panel()
    buttonPanel.setLayout(new HorizontalLayout(10))

    loginButton = new Button("Login")
    loginButton.setOnClick(() -> doLogin())

    cancelButton = new Button("Cancel")
    cancelButton.setOnClick(() -> closeForm())

    buttonPanel.addChild(loginButton)
    buttonPanel.addChild(cancelButton)
    form.addChild(buttonPanel)

    form.setOnSubmit(data -> processLogin(data))

    return form
end method

// The client treats all components uniformly
method renderApplication(root: UIComponent, context: RenderContext)
    // Single call renders entire component tree
    root.render(context)
end method

method handleClick(root: UIComponent, x: Integer, y: Integer)
    // Single call propagates to correct component
    event = new UIEvent(EventType.CLICK, x, y)
    root.handleEvent(event)
end method
```

## Known Uses

- **Java AWT/Swing**: `Component` and `Container` form a composite structure. `Container` is a `Component` that can hold other `Component` objects, including other `Container`s.

- **DOM (Document Object Model)**: HTML/XML documents are trees where `Element` nodes can contain other `Element` nodes and text nodes (leaves).

- **React/Vue/Angular**: Component hierarchies where compound components contain child components. JSX allows nesting components naturally.

- **File Systems**: Directories contain files and other directories. Unix `du` command calculates sizes recursively.

- **Graphics Editors**: Adobe Illustrator, Sketch, and Figma use composite structures for grouping shapes.

- **AST (Abstract Syntax Trees)**: Compilers represent code as trees where expressions can contain sub-expressions.

- **Organization Charts**: Employees and departments form trees where departments contain employees and sub-departments.

- **Menu Systems**: Menus contain menu items and submenus, all handled uniformly.

- **Scene Graphs**: 3D graphics engines (Unity, Unreal) use scene graphs where nodes can have children.

## Related Patterns

- **Chain of Responsibility**: Often used with Composite. Child components can forward events to parents.

- **Decorator**: Shares the recursive composition aspect but has different intent. Decorator adds responsibilities; Composite represents part-whole hierarchies.

- **Flyweight**: Can be used to share leaf nodes in the composite structure when many leaves have identical state.

- **Iterator**: Provides ways to traverse composite structures without exposing their representation.

- **Visitor**: Can apply operations across a composite structure without changing the component classes.

- **Builder**: Can be used to construct complex composite structures step by step.

## When NOT to Use

- **Flat structures**: If you only have a single level of containment (no nesting), Composite adds unnecessary complexity.

- **Different leaf operations**: If different leaf types have completely different operations with no commonality, forcing them into a common interface is awkward.

- **Strong type constraints**: If you need compile-time guarantees about what can contain what (e.g., only certain components in certain containers), Composite's uniform interface works against you.

- **Performance-critical deep trees**: Operations on deep trees can be slow. If you need constant-time operations, a different data structure may be better.

- **When leaves and composites are fundamentally different**: If the operations on containers versus primitives share little in common, a composite interface becomes forced.

- **Simple aggregation**: If you just need a list of objects with no recursive structure, use a simple collection.

- **External tree representation**: If the tree structure is defined externally (database, configuration) and you just need to traverse it, a tree data structure may be simpler than full Composite pattern.

---

## Summary

The Composite pattern is fundamental for representing hierarchical structures where clients need uniform access to both individual objects and groups. Its power comes from the recursive nature - a composite's children can themselves be composites, enabling arbitrarily deep structures. The key design decision is whether to prioritize transparency (all operations in Component) or safety (child operations only in Composite). Choose Composite when you have a clear part-whole hierarchy and need clients to work with it uniformly.

---

*Based on concepts from "Design Patterns: Elements of Reusable Object-Oriented Software" by Gamma, Helm, Johnson, and Vlissides (Gang of Four), 1994.*
