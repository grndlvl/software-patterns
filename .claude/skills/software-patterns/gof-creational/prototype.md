# Prototype

## Intent

Specify the kinds of objects to create using a prototypical instance, and create new objects by copying this prototype. The Prototype pattern allows you to produce new objects by cloning existing ones without coupling your code to their specific classes.

## Also Known As

- Clone
- Cloneable

## Motivation

Consider a graphic editor that supports drawing and manipulating shapes. Users can create circles, rectangles, lines, and complex composite shapes. They can also copy existing shapes to create duplicates. When the user copies a shape, the editor needs to create an exact duplicate—same size, color, position offset, line style, and any other properties.

The challenge is that the editor works with shapes through an abstract Shape interface. When copying a shape, the editor doesn't know (and shouldn't need to know) whether it's a Circle, Rectangle, or ComplexShape. It just knows it has a Shape and needs to create a copy.

One approach would be to check the type of each shape and call the appropriate constructor, but this couples the editor to every concrete shape class and requires updating the code whenever new shape types are added. This violates the Open/Closed Principle.

The Prototype pattern solves this by giving each shape the responsibility of copying itself. Every shape implements a clone() method that creates a duplicate. The editor simply calls clone() on any shape without knowing its concrete type. Each shape knows how to copy its own properties, including any private state that wouldn't be accessible to external code.

This pattern is especially valuable when objects have many properties (some private), when creating objects is expensive (cloning can be cheaper than constructing from scratch), or when the exact types of objects to create are determined at runtime.

## Applicability

Use the Prototype pattern when:

- A system should be independent of how its products are created, composed, and represented
- The classes to instantiate are specified at runtime, for example, by dynamic loading
- You want to avoid building a class hierarchy of factories that parallels the class hierarchy of products
- Instances of a class can have one of only a few different combinations of state (it's more convenient to install a corresponding number of prototypes and clone them rather than instantiating the class manually)
- Creating an object is expensive (database lookup, network call, complex computation), and cloning is cheaper
- Objects have circular references or complex internal structures that are difficult to recreate via constructors
- You need to create copies of objects without coupling to their concrete classes

## Structure

```
┌────────────────────────────────────────────────────────────────────────────┐
│                              CLIENT                                        │
│  Asks a prototype to clone itself                                          │
└────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    │ uses
                                    ▼
┌────────────────────────────────────────────────────────────────────────────┐
│                          «interface»                                       │
│                           PROTOTYPE                                        │
├────────────────────────────────────────────────────────────────────────────┤
│ + clone(): Prototype                                                       │
└────────────────────────────────────────────────────────────────────────────┘
                    ▲                               ▲
                    │ implements                    │ implements
        ┌───────────┴───────────┐       ┌──────────┴────────────┐
        │                       │       │                       │
┌───────┴───────────────┐   ┌───────────┴───────────┐   ┌───────────────────┐
│  ConcretePrototype1   │   │  ConcretePrototype2   │   │ ConcretePrototype3│
├───────────────────────┤   ├───────────────────────┤   ├───────────────────┤
│ - field1: Type        │   │ - fieldA: Type        │   │ - data: Type      │
│ - field2: Type        │   │ - fieldB: Type        │   │ - config: Type    │
├───────────────────────┤   ├───────────────────────┤   ├───────────────────┤
│ + clone(): Prototype  │   │ + clone(): Prototype  │   │ + clone(): Prototype│
│   {                   │   │   {                   │   │   {               │
│     copy = new CP1()  │   │     copy = new CP2()  │   │     copy = new CP3()│
│     copy.field1 =     │   │     copy.fieldA =     │   │     copy.data =   │
│       this.field1     │   │       this.fieldA     │   │       this.data   │
│     copy.field2 =     │   │     copy.fieldB =     │   │     copy.config = │
│       this.field2     │   │       this.fieldB     │   │       this.config │
│     return copy       │   │     return copy       │   │     return copy   │
│   }                   │   │   }                   │   │   }               │
└───────────────────────┘   └───────────────────────┘   └───────────────────┘


PROTOTYPE REGISTRY VARIANT:

┌────────────────────────────────────────────────────────────────────────────┐
│                        PROTOTYPE REGISTRY                                  │
├────────────────────────────────────────────────────────────────────────────┤
│ - prototypes: Map<String, Prototype>                                       │
├────────────────────────────────────────────────────────────────────────────┤
│ + register(key: String, prototype: Prototype): void                        │
│ + unregister(key: String): void                                            │
│ + get(key: String): Prototype   // returns prototype.clone()               │
│ + list(): List<String>                                                     │
└────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    │ stores/clones
                                    ▼
                          ┌─────────────────┐
                          │   Prototype     │
                          └─────────────────┘
```

## Participants

- **Prototype** (Shape, Cloneable)
  - Declares an interface for cloning itself

- **ConcretePrototype** (Circle, Rectangle, ComplexShape)
  - Implements an operation for cloning itself

- **Client** (GraphicEditor)
  - Creates a new object by asking a prototype to clone itself

- **PrototypeRegistry** (ShapeRegistry) [optional]
  - Stores and retrieves prototypes by key
  - Returns clones of stored prototypes on request

## Collaborations

- A client asks a prototype to clone itself. The prototype creates and returns a copy of itself.

- The client doesn't need to know the concrete class of the object it's cloning. It only knows the prototype interface.

- When using a registry, the client requests a clone by key, and the registry returns a clone of the corresponding prototype.

## Consequences

### Benefits

- **Adding and removing products at runtime**: Prototypes let you incorporate a new concrete product class into a system simply by registering a prototypical instance with the client. That's more flexible than other creational patterns, because a client can install and remove prototypes at runtime.

- **Specifying new objects by varying values**: Highly dynamic systems let you define new behavior through object composition—by specifying values for an object's variables—rather than defining new classes. You effectively define new kinds of objects by instantiating existing classes and registering the instances as prototypes of client objects.

- **Specifying new objects by varying structure**: Many applications build objects from parts and subparts. For convenience, such applications often let you instantiate complex, user-defined structures to use a specific subcircuit again and again. The Prototype pattern supports this as well.

- **Reduced subclassing**: Factory Method often produces a hierarchy of Creator classes that parallels the product class hierarchy. The Prototype pattern lets you clone a prototype instead of asking a factory method to make a new object. Hence you don't need a Creator class hierarchy at all.

- **Configuring an application with classes dynamically**: Some runtime environments let you load classes into an application dynamically. The Prototype pattern is key to exploiting such facilities in a language-independent way.

- **Hiding concrete classes from clients**: Clients work with the Prototype interface and don't need to know about concrete implementations.

### Liabilities

- **Complexity of clone implementation**: Implementing clone() can be difficult when object structures contain circular references or when objects contain references to objects that don't support copying.

- **Deep vs. shallow copy decisions**: Each concrete prototype must decide whether to perform deep or shallow copying, and this decision affects all cloned objects.

- **Initialization after cloning**: Sometimes cloned objects need post-clone initialization that can't be done in the clone method itself.

- **Language support varies**: Some languages don't provide good built-in support for cloning objects.

## Implementation

Consider the following implementation issues:

### 1. Deep Copy vs. Shallow Copy

The most critical implementation decision is whether to perform deep or shallow copying:

```pseudocode
class Document implements Prototype {
    private title: String
    private content: String
    private metadata: Metadata      // Reference type
    private sections: List<Section> // Collection of reference types

    // SHALLOW COPY - shares references with original
    function shallowClone(): Document {
        copy = new Document()
        copy.title = this.title           // Strings are typically immutable, OK to share
        copy.content = this.content
        copy.metadata = this.metadata     // DANGER: shares same Metadata object
        copy.sections = this.sections     // DANGER: shares same List
        return copy
    }

    // DEEP COPY - creates independent copies of everything
    function deepClone(): Document {
        copy = new Document()
        copy.title = this.title
        copy.content = this.content
        copy.metadata = this.metadata.clone()  // Clone the metadata
        copy.sections = []
        for (section in this.sections) {
            copy.sections.add(section.clone()) // Clone each section
        }
        return copy
    }

    // Standard clone() - choose appropriate strategy
    function clone(): Document {
        return deepClone()  // Usually deep copy is safer
    }
}
```

### 2. Using a Copy Constructor

Many languages support copy constructors as an alternative to clone methods:

```pseudocode
class Circle implements Shape {
    private centerX: Float
    private centerY: Float
    private radius: Float
    private color: Color
    private lineStyle: LineStyle

    // Regular constructor
    constructor(x: Float, y: Float, radius: Float) {
        this.centerX = x
        this.centerY = y
        this.radius = radius
        this.color = Color.BLACK
        this.lineStyle = LineStyle.SOLID
    }

    // Copy constructor
    constructor(other: Circle) {
        this.centerX = other.centerX
        this.centerY = other.centerY
        this.radius = other.radius
        this.color = other.color.clone()  // Deep copy mutable objects
        this.lineStyle = other.lineStyle  // Enum, safe to share
    }

    function clone(): Shape {
        return new Circle(this)  // Use copy constructor
    }
}
```

### 3. Initializing Clones

Sometimes cloned objects need post-clone initialization:

```pseudocode
interface Prototype {
    function clone(): Prototype
}

interface InitializablePrototype extends Prototype {
    function initialize(params: Map<String, Object>): void
}

class GameCharacter implements InitializablePrototype {
    private name: String
    private health: Integer
    private position: Position
    private inventory: List<Item>
    private id: String

    function clone(): GameCharacter {
        copy = new GameCharacter()
        copy.name = this.name
        copy.health = this.health
        copy.position = this.position.clone()
        copy.inventory = []
        for (item in this.inventory) {
            copy.inventory.add(item.clone())
        }
        // ID is NOT copied - must be unique
        copy.id = null
        return copy
    }

    function initialize(params: Map<String, Object>): void {
        // Set unique ID for the clone
        this.id = UUID.randomUUID().toString()

        // Override any properties from params
        if (params.containsKey("name")) {
            this.name = params.get("name")
        }
        if (params.containsKey("position")) {
            this.position = params.get("position")
        }
    }
}

// Usage
original = loadCharacterTemplate("warrior")
clone = original.clone()
clone.initialize({
    "name": "Player1",
    "position": new Position(100, 200)
})
```

### 4. Prototype Registry

A registry stores prototypes and returns clones:

```pseudocode
class PrototypeRegistry {
    private prototypes: Map<String, Prototype> = {}

    function register(key: String, prototype: Prototype) {
        prototypes.put(key, prototype)
    }

    function unregister(key: String) {
        prototypes.remove(key)
    }

    function get(key: String): Prototype {
        prototype = prototypes.get(key)
        if (prototype == null) {
            throw new PrototypeNotFoundException(key)
        }
        return prototype.clone()
    }

    function exists(key: String): Boolean {
        return prototypes.containsKey(key)
    }

    function listKeys(): List<String> {
        return prototypes.keys().toList()
    }

    // Bulk registration from configuration
    function loadFromConfig(config: PrototypeConfig) {
        for (entry in config.getPrototypes()) {
            prototype = createFromConfig(entry)
            register(entry.key, prototype)
        }
    }
}
```

### 5. Handling Circular References

Objects with circular references require special handling:

```pseudocode
class Node implements Prototype {
    private value: Object
    private children: List<Node>
    private parent: Node  // Circular reference!

    function clone(): Node {
        return cloneWithContext(new CloneContext())
    }

    function cloneWithContext(context: CloneContext): Node {
        // Check if we've already cloned this node
        if (context.hasClone(this)) {
            return context.getClone(this)
        }

        // Create new node and register before cloning children
        copy = new Node()
        context.registerClone(this, copy)

        copy.value = this.value

        // Clone children
        copy.children = []
        for (child in this.children) {
            clonedChild = child.cloneWithContext(context)
            clonedChild.parent = copy  // Set parent to cloned copy
            copy.children.add(clonedChild)
        }

        return copy
    }
}

class CloneContext {
    private cloneMap: Map<Object, Object> = {}

    function hasClone(original: Object): Boolean {
        return cloneMap.containsKey(original)
    }

    function getClone(original: Object): Object {
        return cloneMap.get(original)
    }

    function registerClone(original: Object, clone: Object) {
        cloneMap.put(original, clone)
    }
}
```

### 6. Using Serialization for Deep Copy

In languages with serialization support, you can serialize and deserialize to create deep copies:

```pseudocode
class SerializationCloner {
    static function deepClone<T>(original: T): T {
        // Serialize to bytes
        bytes = Serializer.serialize(original)
        // Deserialize to new object
        return Serializer.deserialize(bytes)
    }
}

class ComplexObject implements Serializable, Prototype {
    private data: Map<String, Object>
    private nested: List<ComplexObject>

    function clone(): ComplexObject {
        return SerializationCloner.deepClone(this)
    }
}
```

### 7. Clone Factory Pattern

Combine Prototype with Factory for more flexibility:

```pseudocode
class CloneFactory<T extends Prototype> {
    private prototype: T

    constructor(prototype: T) {
        this.prototype = prototype
    }

    function create(): T {
        return prototype.clone()
    }

    function createWithModifications(modifier: Consumer<T>): T {
        clone = prototype.clone()
        modifier(clone)
        return clone
    }

    function setPrototype(newPrototype: T) {
        this.prototype = newPrototype
    }
}

// Usage
baseConfig = new ServerConfiguration()
baseConfig.setDefaults()

factory = new CloneFactory(baseConfig)

// Create variations
devConfig = factory.createWithModifications(config => {
    config.setDebugMode(true)
    config.setLogLevel("DEBUG")
})

prodConfig = factory.createWithModifications(config => {
    config.setDebugMode(false)
    config.setLogLevel("ERROR")
})
```

## Example

Here's a complete example of the Prototype pattern applied to a document template system:

```pseudocode
// ============================================================
// PROTOTYPE INTERFACE
// ============================================================

interface DocumentElement {
    function clone(): DocumentElement
    function render(): String
    function getType(): String
}

// ============================================================
// CONCRETE PROTOTYPES
// ============================================================

class TextBlock implements DocumentElement {
    private content: String
    private fontFamily: String
    private fontSize: Integer
    private fontWeight: String
    private color: String
    private alignment: String
    private lineHeight: Float
    private margins: Margins

    constructor() {
        this.content = ""
        this.fontFamily = "Arial"
        this.fontSize = 12
        this.fontWeight = "normal"
        this.color = "#000000"
        this.alignment = "left"
        this.lineHeight = 1.5
        this.margins = new Margins(0, 0, 0, 0)
    }

    // Copy constructor
    constructor(other: TextBlock) {
        this.content = other.content
        this.fontFamily = other.fontFamily
        this.fontSize = other.fontSize
        this.fontWeight = other.fontWeight
        this.color = other.color
        this.alignment = other.alignment
        this.lineHeight = other.lineHeight
        this.margins = other.margins.clone()
    }

    function clone(): DocumentElement {
        return new TextBlock(this)
    }

    function render(): String {
        style = buildStyleString()
        return '<div style="' + style + '">' + escapeHtml(content) + '</div>'
    }

    function getType(): String {
        return "text"
    }

    // Fluent setters for configuration
    function withContent(content: String): TextBlock {
        this.content = content
        return this
    }

    function withFont(family: String, size: Integer): TextBlock {
        this.fontFamily = family
        this.fontSize = size
        return this
    }

    function withBold(): TextBlock {
        this.fontWeight = "bold"
        return this
    }

    function withColor(color: String): TextBlock {
        this.color = color
        return this
    }

    function withAlignment(alignment: String): TextBlock {
        this.alignment = alignment
        return this
    }

    private function buildStyleString(): String {
        return "font-family: " + fontFamily + "; " +
               "font-size: " + fontSize + "px; " +
               "font-weight: " + fontWeight + "; " +
               "color: " + color + "; " +
               "text-align: " + alignment + "; " +
               "line-height: " + lineHeight + ";"
    }
}

class ImageBlock implements DocumentElement {
    private source: String
    private altText: String
    private width: Integer
    private height: Integer
    private alignment: String
    private caption: String
    private border: BorderStyle
    private linkUrl: String

    constructor() {
        this.source = ""
        this.altText = ""
        this.width = 0
        this.height = 0
        this.alignment = "left"
        this.caption = ""
        this.border = null
        this.linkUrl = null
    }

    constructor(other: ImageBlock) {
        this.source = other.source
        this.altText = other.altText
        this.width = other.width
        this.height = other.height
        this.alignment = other.alignment
        this.caption = other.caption
        this.border = other.border != null ? other.border.clone() : null
        this.linkUrl = other.linkUrl
    }

    function clone(): DocumentElement {
        return new ImageBlock(this)
    }

    function render(): String {
        img = '<img src="' + source + '" alt="' + altText + '"'
        if (width > 0) img += ' width="' + width + '"'
        if (height > 0) img += ' height="' + height + '"'
        if (border != null) img += ' style="' + border.toCSS() + '"'
        img += ' />'

        if (linkUrl != null) {
            img = '<a href="' + linkUrl + '">' + img + '</a>'
        }

        result = '<figure style="text-align: ' + alignment + '">' + img
        if (caption != "") {
            result += '<figcaption>' + caption + '</figcaption>'
        }
        result += '</figure>'

        return result
    }

    function getType(): String {
        return "image"
    }

    function withSource(source: String): ImageBlock {
        this.source = source
        return this
    }

    function withDimensions(width: Integer, height: Integer): ImageBlock {
        this.width = width
        this.height = height
        return this
    }

    function withCaption(caption: String): ImageBlock {
        this.caption = caption
        return this
    }

    function withLink(url: String): ImageBlock {
        this.linkUrl = url
        return this
    }
}

class TableBlock implements DocumentElement {
    private headers: List<String>
    private rows: List<List<String>>
    private columnWidths: List<Integer>
    private headerStyle: CellStyle
    private cellStyle: CellStyle
    private alternateRowColor: String
    private bordered: Boolean

    constructor() {
        this.headers = []
        this.rows = []
        this.columnWidths = []
        this.headerStyle = new CellStyle()
        this.cellStyle = new CellStyle()
        this.alternateRowColor = null
        this.bordered = true
    }

    constructor(other: TableBlock) {
        this.headers = List.copyOf(other.headers)
        this.rows = []
        for (row in other.rows) {
            this.rows.add(List.copyOf(row))
        }
        this.columnWidths = List.copyOf(other.columnWidths)
        this.headerStyle = other.headerStyle.clone()
        this.cellStyle = other.cellStyle.clone()
        this.alternateRowColor = other.alternateRowColor
        this.bordered = other.bordered
    }

    function clone(): DocumentElement {
        return new TableBlock(this)
    }

    function render(): String {
        borderAttr = bordered ? ' border="1"' : ''
        result = '<table' + borderAttr + '>'

        if (headers.size() > 0) {
            result += '<thead><tr>'
            for (i = 0; i < headers.size(); i++) {
                width = i < columnWidths.size() ? columnWidths.get(i) : 0
                style = headerStyle.toCSS()
                if (width > 0) style += "width: " + width + "px;"
                result += '<th style="' + style + '">' + headers.get(i) + '</th>'
            }
            result += '</tr></thead>'
        }

        result += '<tbody>'
        for (rowIndex = 0; rowIndex < rows.size(); rowIndex++) {
            row = rows.get(rowIndex)
            rowStyle = cellStyle.toCSS()
            if (alternateRowColor != null && rowIndex % 2 == 1) {
                rowStyle += "background-color: " + alternateRowColor + ";"
            }
            result += '<tr>'
            for (cell in row) {
                result += '<td style="' + rowStyle + '">' + cell + '</td>'
            }
            result += '</tr>'
        }
        result += '</tbody></table>'

        return result
    }

    function getType(): String {
        return "table"
    }

    function withHeaders(headers: List<String>): TableBlock {
        this.headers = headers
        return this
    }

    function addRow(cells: List<String>): TableBlock {
        this.rows.add(cells)
        return this
    }

    function withAlternateRowColor(color: String): TableBlock {
        this.alternateRowColor = color
        return this
    }

    function withoutBorder(): TableBlock {
        this.bordered = false
        return this
    }
}

class CompositeBlock implements DocumentElement {
    private children: List<DocumentElement>
    private layout: String  // "vertical", "horizontal", "grid"
    private spacing: Integer
    private backgroundColor: String
    private padding: Integer
    private border: BorderStyle

    constructor() {
        this.children = []
        this.layout = "vertical"
        this.spacing = 10
        this.backgroundColor = null
        this.padding = 0
        this.border = null
    }

    constructor(other: CompositeBlock) {
        this.children = []
        for (child in other.children) {
            this.children.add(child.clone())  // Deep clone children
        }
        this.layout = other.layout
        this.spacing = other.spacing
        this.backgroundColor = other.backgroundColor
        this.padding = other.padding
        this.border = other.border != null ? other.border.clone() : null
    }

    function clone(): DocumentElement {
        return new CompositeBlock(this)
    }

    function render(): String {
        style = buildContainerStyle()
        result = '<div style="' + style + '">'
        for (child in children) {
            result += '<div style="margin-bottom: ' + spacing + 'px">'
            result += child.render()
            result += '</div>'
        }
        result += '</div>'
        return result
    }

    function getType(): String {
        return "composite"
    }

    function addChild(child: DocumentElement): CompositeBlock {
        this.children.add(child)
        return this
    }

    function withLayout(layout: String): CompositeBlock {
        this.layout = layout
        return this
    }

    function withSpacing(spacing: Integer): CompositeBlock {
        this.spacing = spacing
        return this
    }

    function withBackgroundColor(color: String): CompositeBlock {
        this.backgroundColor = color
        return this
    }

    private function buildContainerStyle(): String {
        style = ""
        if (layout == "horizontal") {
            style += "display: flex; flex-direction: row; "
        } else if (layout == "grid") {
            style += "display: grid; "
        }
        if (backgroundColor != null) {
            style += "background-color: " + backgroundColor + "; "
        }
        if (padding > 0) {
            style += "padding: " + padding + "px; "
        }
        return style
    }
}

// ============================================================
// PROTOTYPE REGISTRY
// ============================================================

class DocumentElementRegistry {
    private static instance: DocumentElementRegistry = null
    private prototypes: Map<String, DocumentElement> = {}

    private constructor() {
        initializeDefaults()
    }

    static function getInstance(): DocumentElementRegistry {
        if (instance == null) {
            instance = new DocumentElementRegistry()
        }
        return instance
    }

    private function initializeDefaults() {
        // Register default text styles
        heading1 = new TextBlock()
            .withFont("Georgia", 32)
            .withBold()
            .withColor("#333333")
        register("heading1", heading1)

        heading2 = new TextBlock()
            .withFont("Georgia", 24)
            .withBold()
            .withColor("#444444")
        register("heading2", heading2)

        paragraph = new TextBlock()
            .withFont("Arial", 14)
            .withColor("#666666")
        register("paragraph", paragraph)

        caption = new TextBlock()
            .withFont("Arial", 12)
            .withColor("#888888")
            .withAlignment("center")
        register("caption", caption)

        // Register default table style
        dataTable = new TableBlock()
            .withAlternateRowColor("#f5f5f5")
        register("data-table", dataTable)

        // Register default image styles
        thumbnailImage = new ImageBlock()
            .withDimensions(150, 150)
        register("thumbnail", thumbnailImage)

        fullWidthImage = new ImageBlock()
            .withDimensions(800, 0)  // 0 height = auto
            .withAlignment("center")
        register("full-width-image", fullWidthImage)

        // Register composite templates
        articleSection = new CompositeBlock()
            .withSpacing(20)
            .withBackgroundColor("#ffffff")
        register("article-section", articleSection)

        sidebar = new CompositeBlock()
            .withLayout("vertical")
            .withSpacing(15)
            .withBackgroundColor("#f9f9f9")
        register("sidebar", sidebar)
    }

    function register(key: String, prototype: DocumentElement) {
        prototypes.put(key, prototype)
    }

    function unregister(key: String) {
        prototypes.remove(key)
    }

    function get(key: String): DocumentElement {
        prototype = prototypes.get(key)
        if (prototype == null) {
            throw new UnknownPrototypeException(
                "No prototype registered with key: " + key
            )
        }
        return prototype.clone()
    }

    function getAll(): Map<String, DocumentElement> {
        result = {}
        for ((key, prototype) in prototypes) {
            result.put(key, prototype.clone())
        }
        return result
    }

    function listKeys(): List<String> {
        return prototypes.keys().toList().sorted()
    }

    function exists(key: String): Boolean {
        return prototypes.containsKey(key)
    }
}

// ============================================================
// DOCUMENT BUILDER USING PROTOTYPES
// ============================================================

class Document {
    private title: String
    private elements: List<DocumentElement>
    private metadata: Map<String, String>

    constructor(title: String) {
        this.title = title
        this.elements = []
        this.metadata = {}
    }

    function addElement(element: DocumentElement) {
        elements.add(element)
    }

    function setMetadata(key: String, value: String) {
        metadata.put(key, value)
    }

    function render(): String {
        html = "<!DOCTYPE html><html><head>"
        html += "<title>" + title + "</title>"
        html += "<meta charset='UTF-8'>"
        for ((key, value) in metadata) {
            html += '<meta name="' + key + '" content="' + value + '">'
        }
        html += "</head><body>"
        for (element in elements) {
            html += element.render()
        }
        html += "</body></html>"
        return html
    }
}

class DocumentBuilder {
    private registry: DocumentElementRegistry
    private document: Document

    constructor(title: String) {
        this.registry = DocumentElementRegistry.getInstance()
        this.document = new Document(title)
    }

    function addHeading(level: Integer, text: String): DocumentBuilder {
        key = "heading" + level
        if (!registry.exists(key)) {
            key = "heading1"  // Fallback
        }
        heading = registry.get(key) as TextBlock
        heading.withContent(text)
        document.addElement(heading)
        return this
    }

    function addParagraph(text: String): DocumentBuilder {
        paragraph = registry.get("paragraph") as TextBlock
        paragraph.withContent(text)
        document.addElement(paragraph)
        return this
    }

    function addImage(source: String, caption: String): DocumentBuilder {
        image = registry.get("full-width-image") as ImageBlock
        image.withSource(source).withCaption(caption)
        document.addElement(image)
        return this
    }

    function addThumbnail(source: String): DocumentBuilder {
        image = registry.get("thumbnail") as ImageBlock
        image.withSource(source)
        document.addElement(image)
        return this
    }

    function addDataTable(headers: List<String>, data: List<List<String>>): DocumentBuilder {
        table = registry.get("data-table") as TableBlock
        table.withHeaders(headers)
        for (row in data) {
            table.addRow(row)
        }
        document.addElement(table)
        return this
    }

    function addCustomElement(prototypeKey: String, configurator: Consumer<DocumentElement>): DocumentBuilder {
        element = registry.get(prototypeKey)
        configurator(element)
        document.addElement(element)
        return this
    }

    function build(): Document {
        return document
    }
}

// ============================================================
// USAGE EXAMPLES
// ============================================================

function main() {
    // Register custom prototypes for this application
    registry = DocumentElementRegistry.getInstance()

    // Custom styled quote block
    quoteBlock = new TextBlock()
        .withFont("Georgia", 18)
        .withColor("#555555")
        .withAlignment("center")
    registry.register("quote", quoteBlock)

    // Custom styled code block
    codeBlock = new TextBlock()
        .withFont("Consolas", 14)
        .withColor("#333333")
    registry.register("code", codeBlock)

    // Build a document using prototypes
    document = new DocumentBuilder("Quarterly Report")
        .addHeading(1, "Q3 2024 Performance Report")
        .addParagraph("This report summarizes our performance metrics for the third quarter.")
        .addImage("/images/chart.png", "Revenue Growth Chart")
        .addHeading(2, "Sales by Region")
        .addDataTable(
            ["Region", "Revenue", "Growth"],
            [
                ["North America", "$2.5M", "+15%"],
                ["Europe", "$1.8M", "+12%"],
                ["Asia Pacific", "$1.2M", "+28%"]
            ]
        )
        .addCustomElement("quote", element => {
            (element as TextBlock).withContent(
                "Our team exceeded expectations this quarter."
            )
        })
        .build()

    // Render to HTML
    html = document.render()
    File.write("report.html", html)

    // Clone an element and modify it
    original = registry.get("paragraph") as TextBlock
    modified = original.clone() as TextBlock
    modified.withContent("This is cloned and modified content.")

    // Elements are independent
    print(original.render())   // Original content
    print(modified.render())   // Modified content
}
```

## Known Uses

- **Java's Object.clone()**: The canonical example of the Prototype pattern. Classes implement Cloneable and override clone().

- **JavaScript Object Spread/Assign**: `{...obj}` and `Object.assign()` create shallow copies of objects.

- **Python's copy module**: `copy.copy()` and `copy.deepcopy()` implement shallow and deep cloning.

- **Game Development**: Character templates, enemy types, and level prefabs are typically implemented as prototypes that get cloned and customized.

- **Document Editors**: Microsoft Word, Google Docs, and similar tools use prototypes for formatting styles and templates.

- **GUI Toolkits**: Widget templates and style presets are often implemented as prototypes.

- **.NET's ICloneable interface**: Similar to Java's Cloneable.

- **Database Connection Pooling**: Connection pools often clone a prototype connection configuration.

- **Spreadsheet Applications**: Cell formatting presets and chart templates.

- **3D Modeling Software**: Object instancing in Blender, Maya, etc., where duplicates share mesh data.

## Related Patterns

- **Abstract Factory**: Abstract Factory might use Prototype to create products. Instead of creating products via new, the factory clones prototypes.

- **Factory Method**: Factory Method is based on inheritance and relies on a subclass to handle the desired object creation. Prototype doesn't require subclassing, but it does require an "initialize" operation.

- **Composite**: Designs that make heavy use of Composite and Decorator often benefit from Prototype as well. Applying the pattern lets you clone complex structures instead of re-creating them from scratch.

- **Memento**: Can be used with Prototype to store and restore object state. A memento can be implemented as a prototype.

- **Singleton**: Prototype and Singleton are often used together. A Singleton can store the prototype registry.

## When NOT to Use

- **Simple objects**: If objects have simple constructors with few parameters and no complex initialization, direct instantiation is clearer.

- **Immutable objects**: Immutable objects don't need cloning—you can safely share references.

- **No runtime variation needed**: If object types are known at compile time and don't change, factory methods are simpler.

- **Cloning is expensive**: If objects hold resources that can't be easily copied (network connections, file handles), cloning may not be practical.

- **Object identity matters**: If each object needs a unique identity that shouldn't be copied, Prototype can cause problems.

**Simpler alternatives**:

- **Copy constructors**: For objects with straightforward copying needs
  ```
  newObj = new MyClass(existingObj)
  ```

- **Factory methods**: When you have a known set of object configurations
  ```
  config = ConfigFactory.createDevelopmentConfig()
  ```

- **Builder pattern**: When objects need step-by-step construction with variations
  ```
  config = ConfigBuilder.withDefaults().setDebug(true).build()
  ```

- **Object spread/destructuring** (JavaScript):
  ```
  newObj = { ...existingObj, modifiedProperty: newValue }
  ```

**Signs you've over-engineered**:
- Your clone() method just calls the constructor with the same arguments
- You never actually clone objects at runtime—all types are known at compile time
- The "prototypes" in your registry are never modified before use
- You're cloning simple value objects that could be created directly
- Your objects don't have any private state that benefits from clone access

---

*Based on concepts from "Design Patterns: Elements of Reusable Object-Oriented Software" by Gamma, Helm, Johnson, and Vlissides (Gang of Four), 1994.*
