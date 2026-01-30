# Abstract Factory

## Intent

Provide an interface for creating families of related or dependent objects without specifying their concrete classes. The Abstract Factory pattern lets you produce objects that belong together (a "family") while ensuring that incompatible objects from different families cannot be mixed.

## Also Known As

- Kit
- Toolkit

## Motivation

Consider a user interface toolkit that supports multiple look-and-feel standards, such as a native desktop appearance, a flat modern design, and a high-contrast accessibility theme. Each look-and-feel defines different appearances and behaviors for user interface widgets like buttons, text fields, and scrollbars. To be portable across look-and-feel standards, an application should not hard-code its widgets for a particular look and feel—instantiating look-and-feel-specific classes of widgets throughout the application makes it difficult to change the look and feel later.

We can solve this problem by defining an abstract WidgetFactory class that declares an interface for creating each basic kind of widget. There's also an abstract class for each kind of widget, and concrete subclasses implement widgets for specific look-and-feel standards. WidgetFactory's interface has an operation that returns a new widget object for each abstract widget class. Clients call these operations to obtain widget instances, but clients aren't aware of the concrete classes they're using. Thus clients stay independent of the prevailing look and feel.

For each look-and-feel standard, there is a concrete subclass of WidgetFactory. Each subclass implements the operations to create the appropriate widget for the look and feel. For example, the CreateButton operation on the MaterialWidgetFactory instantiates and returns a Material button, while the corresponding operation on the NativeWidgetFactory returns a native desktop button. Clients create widgets solely through the WidgetFactory interface and have no knowledge of the classes that implement widgets for a particular look and feel. In other words, clients only have to commit to an interface defined by an abstract class, not a particular concrete class.

## Applicability

Use the Abstract Factory pattern when:

- A system should be independent of how its products are created, composed, and represented
- A system should be configured with one of multiple families of products
- A family of related product objects is designed to be used together, and you need to enforce this constraint
- You want to provide a class library of products, and you want to reveal just their interfaces, not their implementations
- You need to ensure that products from one family are not mixed with products from another family
- You want to add new product families without changing existing client code

## Structure

```
┌───────────────────────────────────────────────────────────────────────────────┐
│                              CLIENT                                           │
│  Uses only interfaces declared by AbstractFactory and AbstractProduct        │
└───────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    │ uses
                                    ▼
┌───────────────────────────────────────────────────────────────────────────────┐
│                          «interface»                                          │
│                        AbstractFactory                                        │
├───────────────────────────────────────────────────────────────────────────────┤
│ + createProductA(): AbstractProductA                                          │
│ + createProductB(): AbstractProductB                                          │
└───────────────────────────────────────────────────────────────────────────────┘
                    ▲                               ▲
                    │ implements                    │ implements
        ┌───────────┴───────────┐       ┌──────────┴────────────┐
        │                       │       │                       │
┌───────┴───────────┐   ┌───────┴───────────┐   ┌───────────────┴───┐
│ ConcreteFactory1  │   │ ConcreteFactory2  │   │ ConcreteFactory3  │
├───────────────────┤   ├───────────────────┤   ├───────────────────┤
│ +createProductA() │   │ +createProductA() │   │ +createProductA() │
│ +createProductB() │   │ +createProductB() │   │ +createProductB() │
└───────────────────┘   └───────────────────┘   └───────────────────┘
        │                       │                       │
        │ creates               │ creates               │ creates
        ▼                       ▼                       ▼
┌───────────────────────────────────────────────────────────────────────────────┐
│                          «interface»                                          │
│                        AbstractProductA                                       │
└───────────────────────────────────────────────────────────────────────────────┘
                    ▲                               ▲
                    │ implements                    │ implements
        ┌───────────┴───────────┐       ┌──────────┴────────────┐
        │                       │       │                       │
┌───────┴───────────┐   ┌───────┴───────────┐   ┌───────────────┴───┐
│ ConcreteProductA1 │   │ ConcreteProductA2 │   │ ConcreteProductA3 │
└───────────────────┘   └───────────────────┘   └───────────────────┘

┌───────────────────────────────────────────────────────────────────────────────┐
│                          «interface»                                          │
│                        AbstractProductB                                       │
└───────────────────────────────────────────────────────────────────────────────┘
                    ▲                               ▲
                    │ implements                    │ implements
        ┌───────────┴───────────┐       ┌──────────┴────────────┐
        │                       │       │                       │
┌───────┴───────────┐   ┌───────┴───────────┐   ┌───────────────┴───┐
│ ConcreteProductB1 │   │ ConcreteProductB2 │   │ ConcreteProductB3 │
└───────────────────┘   └───────────────────┘   └───────────────────┘
```

## Participants

- **AbstractFactory** (WidgetFactory)
  - Declares an interface for operations that create abstract product objects

- **ConcreteFactory** (MaterialWidgetFactory, NativeWidgetFactory, AccessibilityWidgetFactory)
  - Implements the operations to create concrete product objects
  - Each concrete factory creates products belonging to a single family

- **AbstractProduct** (Button, TextField, Scrollbar)
  - Declares an interface for a type of product object

- **ConcreteProduct** (MaterialButton, NativeButton, AccessibilityButton)
  - Defines a product object to be created by the corresponding concrete factory
  - Implements the AbstractProduct interface

- **Client**
  - Uses only interfaces declared by AbstractFactory and AbstractProduct classes
  - Never references concrete product or factory classes directly

## Collaborations

- Normally a single instance of a ConcreteFactory class is created at runtime. This concrete factory creates product objects having a particular implementation. To create different product objects, clients should use a different concrete factory.

- AbstractFactory defers creation of product objects to its ConcreteFactory subclass.

- The client manipulates instances through their abstract interfaces. Product class names are isolated in the implementation of the concrete factory; they do not appear in client code.

## Consequences

### Benefits

- **Isolation of concrete classes**: The Abstract Factory pattern helps you control the classes of objects that an application creates. Because a factory encapsulates the responsibility and the process of creating product objects, it isolates clients from implementation classes. Clients manipulate instances through their abstract interfaces. Product class names are isolated in the implementation of the concrete factory; they do not appear in client code.

- **Exchanging product families easily**: The class of a concrete factory appears only once in an application—where it's instantiated. This makes it easy to change the concrete factory an application uses. It can use different product configurations simply by changing the concrete factory. Because an abstract factory creates a complete family of products, the whole product family changes at once.

- **Promoting consistency among products**: When product objects in a family are designed to work together, it's important that an application use objects from only one family at a time. AbstractFactory makes this easy to enforce.

- **Supporting new kinds of products**: Extending abstract factories to produce new kinds of Products isn't easy. That's because the AbstractFactory interface fixes the set of products that can be created. Supporting new kinds of products requires extending the factory interface, which involves changing the AbstractFactory class and all of its subclasses.

### Liabilities

- **Difficulty supporting new kinds of products**: The AbstractFactory interface fixes the set of products that can be created. Supporting new kinds of products requires extending the factory interface. This involves changing the AbstractFactory class and all of its subclasses.

- **Increased complexity**: The pattern introduces many new interfaces and classes. For small applications with few product families, this overhead may not be justified.

- **All products must be created through the factory**: Clients cannot create products directly. This can be limiting if some products don't need the full abstraction.

## Implementation

Consider the following implementation issues:

### 1. Factories as Singletons

An application typically needs only one instance of a ConcreteFactory per product family. So it's usually best implemented as a Singleton.

```pseudocode
class ConcreteFactory1 extends AbstractFactory {
    private static instance: ConcreteFactory1 = null

    private constructor() {
        // Private constructor prevents external instantiation
    }

    static function getInstance(): ConcreteFactory1 {
        if (instance == null) {
            instance = new ConcreteFactory1()
        }
        return instance
    }

    function createProductA(): AbstractProductA {
        return new ConcreteProductA1()
    }

    function createProductB(): AbstractProductB {
        return new ConcreteProductB1()
    }
}
```

### 2. Creating the Products

AbstractFactory only declares an interface for creating products. It's up to ConcreteProduct subclasses to actually create them. The most common way to do this is to define a factory method for each product. A concrete factory will specify its products by overriding the factory method for each.

```pseudocode
interface AbstractFactory {
    function createButton(): Button
    function createTextField(): TextField
    function createCheckbox(): Checkbox
}

class MaterialFactory implements AbstractFactory {
    function createButton(): Button {
        return new MaterialButton()
    }

    function createTextField(): TextField {
        return new MaterialTextField()
    }

    function createCheckbox(): Checkbox {
        return new MaterialCheckbox()
    }
}

class NativeFactory implements AbstractFactory {
    function createButton(): Button {
        return new NativeButton()
    }

    function createTextField(): TextField {
        return new NativeTextField()
    }

    function createCheckbox(): Checkbox {
        return new NativeCheckbox()
    }
}
```

### 3. Defining Extensible Factories

AbstractFactory usually defines a different operation for each kind of product it can produce. Adding a new kind of product requires changing the AbstractFactory interface and all the classes that depend on it. A more flexible but less safe design is to add a parameter to operations that create objects. This parameter specifies the kind of object to be created.

```pseudocode
interface AbstractFactory {
    function createWidget(type: WidgetType): Widget
}

enum WidgetType {
    BUTTON,
    TEXT_FIELD,
    CHECKBOX,
    SLIDER
}

class MaterialFactory implements AbstractFactory {
    function createWidget(type: WidgetType): Widget {
        switch (type) {
            case BUTTON:
                return new MaterialButton()
            case TEXT_FIELD:
                return new MaterialTextField()
            case CHECKBOX:
                return new MaterialCheckbox()
            case SLIDER:
                return new MaterialSlider()
            default:
                throw new UnsupportedWidgetException(type)
        }
    }
}
```

### 4. Using Configuration to Select Factory

```pseudocode
class FactoryProvider {
    private static factories: Map<String, AbstractFactory> = {
        "material": new MaterialFactory(),
        "native": new NativeFactory(),
        "accessibility": new AccessibilityFactory()
    }

    static function getFactory(theme: String): AbstractFactory {
        factory = factories.get(theme.toLowerCase())
        if (factory == null) {
            throw new UnknownThemeException(theme)
        }
        return factory
    }

    static function getFactoryFromConfig(): AbstractFactory {
        theme = Configuration.get("ui.theme", "material")
        return getFactory(theme)
    }
}
```

## Example

Here's a complete example of the Abstract Factory pattern applied to a cross-platform document processing system that can generate documents in different formats (HTML, PDF, Markdown):

```pseudocode
// ============================================================
// ABSTRACT PRODUCTS
// ============================================================

interface DocumentElement {
    function render(): String
}

interface Heading extends DocumentElement {
    function setLevel(level: Integer)
    function setText(text: String)
}

interface Paragraph extends DocumentElement {
    function setText(text: String)
    function setBold(bold: Boolean)
    function setItalic(italic: Boolean)
}

interface Link extends DocumentElement {
    function setUrl(url: String)
    function setText(text: String)
}

interface Image extends DocumentElement {
    function setSource(src: String)
    function setAltText(alt: String)
    function setWidth(width: Integer)
    function setHeight(height: Integer)
}

interface Table extends DocumentElement {
    function setHeaders(headers: List<String>)
    function addRow(cells: List<String>)
}

// ============================================================
// ABSTRACT FACTORY
// ============================================================

interface DocumentFactory {
    function createHeading(): Heading
    function createParagraph(): Paragraph
    function createLink(): Link
    function createImage(): Image
    function createTable(): Table
    function getFileExtension(): String
}

// ============================================================
// CONCRETE PRODUCTS - HTML FAMILY
// ============================================================

class HtmlHeading implements Heading {
    private level: Integer = 1
    private text: String = ""

    function setLevel(level: Integer) {
        this.level = clamp(level, 1, 6)
    }

    function setText(text: String) {
        this.text = escapeHtml(text)
    }

    function render(): String {
        return "<h" + level + ">" + text + "</h" + level + ">"
    }
}

class HtmlParagraph implements Paragraph {
    private text: String = ""
    private bold: Boolean = false
    private italic: Boolean = false

    function setText(text: String) {
        this.text = escapeHtml(text)
    }

    function setBold(bold: Boolean) {
        this.bold = bold
    }

    function setItalic(italic: Boolean) {
        this.italic = italic
    }

    function render(): String {
        content = text
        if (bold) {
            content = "<strong>" + content + "</strong>"
        }
        if (italic) {
            content = "<em>" + content + "</em>"
        }
        return "<p>" + content + "</p>"
    }
}

class HtmlLink implements Link {
    private url: String = ""
    private text: String = ""

    function setUrl(url: String) {
        this.url = escapeHtml(url)
    }

    function setText(text: String) {
        this.text = escapeHtml(text)
    }

    function render(): String {
        return '<a href="' + url + '">' + text + '</a>'
    }
}

class HtmlImage implements Image {
    private src: String = ""
    private alt: String = ""
    private width: Integer = 0
    private height: Integer = 0

    function setSource(src: String) {
        this.src = escapeHtml(src)
    }

    function setAltText(alt: String) {
        this.alt = escapeHtml(alt)
    }

    function setWidth(width: Integer) {
        this.width = width
    }

    function setHeight(height: Integer) {
        this.height = height
    }

    function render(): String {
        result = '<img src="' + src + '" alt="' + alt + '"'
        if (width > 0) {
            result += ' width="' + width + '"'
        }
        if (height > 0) {
            result += ' height="' + height + '"'
        }
        result += ' />'
        return result
    }
}

class HtmlTable implements Table {
    private headers: List<String> = []
    private rows: List<List<String>> = []

    function setHeaders(headers: List<String>) {
        this.headers = headers.map(h => escapeHtml(h))
    }

    function addRow(cells: List<String>) {
        this.rows.add(cells.map(c => escapeHtml(c)))
    }

    function render(): String {
        result = "<table>\n"

        if (headers.length > 0) {
            result += "  <thead>\n    <tr>\n"
            for (header in headers) {
                result += "      <th>" + header + "</th>\n"
            }
            result += "    </tr>\n  </thead>\n"
        }

        result += "  <tbody>\n"
        for (row in rows) {
            result += "    <tr>\n"
            for (cell in row) {
                result += "      <td>" + cell + "</td>\n"
            }
            result += "    </tr>\n"
        }
        result += "  </tbody>\n</table>"

        return result
    }
}

// ============================================================
// CONCRETE PRODUCTS - MARKDOWN FAMILY
// ============================================================

class MarkdownHeading implements Heading {
    private level: Integer = 1
    private text: String = ""

    function setLevel(level: Integer) {
        this.level = clamp(level, 1, 6)
    }

    function setText(text: String) {
        this.text = text
    }

    function render(): String {
        prefix = "#".repeat(level)
        return prefix + " " + text
    }
}

class MarkdownParagraph implements Paragraph {
    private text: String = ""
    private bold: Boolean = false
    private italic: Boolean = false

    function setText(text: String) {
        this.text = text
    }

    function setBold(bold: Boolean) {
        this.bold = bold
    }

    function setItalic(italic: Boolean) {
        this.italic = italic
    }

    function render(): String {
        content = text
        if (bold && italic) {
            content = "***" + content + "***"
        } else if (bold) {
            content = "**" + content + "**"
        } else if (italic) {
            content = "*" + content + "*"
        }
        return content + "\n"
    }
}

class MarkdownLink implements Link {
    private url: String = ""
    private text: String = ""

    function setUrl(url: String) {
        this.url = url
    }

    function setText(text: String) {
        this.text = text
    }

    function render(): String {
        return "[" + text + "](" + url + ")"
    }
}

class MarkdownImage implements Image {
    private src: String = ""
    private alt: String = ""
    private width: Integer = 0
    private height: Integer = 0

    function setSource(src: String) {
        this.src = src
    }

    function setAltText(alt: String) {
        this.alt = alt
    }

    function setWidth(width: Integer) {
        this.width = width
    }

    function setHeight(height: Integer) {
        this.height = height
    }

    function render(): String {
        // Basic Markdown doesn't support dimensions, but some flavors do
        return "![" + alt + "](" + src + ")"
    }
}

class MarkdownTable implements Table {
    private headers: List<String> = []
    private rows: List<List<String>> = []

    function setHeaders(headers: List<String>) {
        this.headers = headers
    }

    function addRow(cells: List<String>) {
        this.rows.add(cells)
    }

    function render(): String {
        if (headers.length == 0) {
            return ""
        }

        result = "| " + headers.join(" | ") + " |\n"
        result += "| " + headers.map(h => "---").join(" | ") + " |\n"

        for (row in rows) {
            result += "| " + row.join(" | ") + " |\n"
        }

        return result
    }
}

// ============================================================
// CONCRETE PRODUCTS - PDF FAMILY (using PDF commands)
// ============================================================

class PdfHeading implements Heading {
    private level: Integer = 1
    private text: String = ""

    function setLevel(level: Integer) {
        this.level = clamp(level, 1, 6)
    }

    function setText(text: String) {
        this.text = text
    }

    function render(): String {
        fontSize = 24 - ((level - 1) * 2)  // h1=24pt, h2=22pt, etc.
        return "PDF_SET_FONT(Helvetica-Bold, " + fontSize + ")\n" +
               "PDF_DRAW_TEXT(" + text + ")\n" +
               "PDF_LINE_BREAK()"
    }
}

class PdfParagraph implements Paragraph {
    private text: String = ""
    private bold: Boolean = false
    private italic: Boolean = false

    function setText(text: String) {
        this.text = text
    }

    function setBold(bold: Boolean) {
        this.bold = bold
    }

    function setItalic(italic: Boolean) {
        this.italic = italic
    }

    function render(): String {
        fontName = "Helvetica"
        if (bold && italic) {
            fontName = "Helvetica-BoldOblique"
        } else if (bold) {
            fontName = "Helvetica-Bold"
        } else if (italic) {
            fontName = "Helvetica-Oblique"
        }
        return "PDF_SET_FONT(" + fontName + ", 12)\n" +
               "PDF_DRAW_TEXT(" + text + ")\n" +
               "PDF_LINE_BREAK()"
    }
}

class PdfLink implements Link {
    private url: String = ""
    private text: String = ""

    function setUrl(url: String) {
        this.url = url
    }

    function setText(text: String) {
        this.text = text
    }

    function render(): String {
        return "PDF_SET_FONT(Helvetica, 12)\n" +
               "PDF_SET_COLOR(0, 0, 255)\n" +
               "PDF_DRAW_LINK(" + text + ", " + url + ")\n" +
               "PDF_SET_COLOR(0, 0, 0)"
    }
}

class PdfImage implements Image {
    private src: String = ""
    private alt: String = ""
    private width: Integer = 0
    private height: Integer = 0

    function setSource(src: String) {
        this.src = src
    }

    function setAltText(alt: String) {
        this.alt = alt
    }

    function setWidth(width: Integer) {
        this.width = width
    }

    function setHeight(height: Integer) {
        this.height = height
    }

    function render(): String {
        return "PDF_DRAW_IMAGE(" + src + ", " + width + ", " + height + ")\n" +
               "PDF_LINE_BREAK()"
    }
}

class PdfTable implements Table {
    private headers: List<String> = []
    private rows: List<List<String>> = []

    function setHeaders(headers: List<String>) {
        this.headers = headers
    }

    function addRow(cells: List<String>) {
        this.rows.add(cells)
    }

    function render(): String {
        colCount = headers.length
        result = "PDF_BEGIN_TABLE(" + colCount + ")\n"

        result += "PDF_SET_FONT(Helvetica-Bold, 10)\n"
        for (header in headers) {
            result += "PDF_TABLE_CELL(" + header + ")\n"
        }
        result += "PDF_TABLE_ROW_END()\n"

        result += "PDF_SET_FONT(Helvetica, 10)\n"
        for (row in rows) {
            for (cell in row) {
                result += "PDF_TABLE_CELL(" + cell + ")\n"
            }
            result += "PDF_TABLE_ROW_END()\n"
        }

        result += "PDF_END_TABLE()"
        return result
    }
}

// ============================================================
// CONCRETE FACTORIES
// ============================================================

class HtmlDocumentFactory implements DocumentFactory {
    function createHeading(): Heading {
        return new HtmlHeading()
    }

    function createParagraph(): Paragraph {
        return new HtmlParagraph()
    }

    function createLink(): Link {
        return new HtmlLink()
    }

    function createImage(): Image {
        return new HtmlImage()
    }

    function createTable(): Table {
        return new HtmlTable()
    }

    function getFileExtension(): String {
        return ".html"
    }
}

class MarkdownDocumentFactory implements DocumentFactory {
    function createHeading(): Heading {
        return new MarkdownHeading()
    }

    function createParagraph(): Paragraph {
        return new MarkdownParagraph()
    }

    function createLink(): Link {
        return new MarkdownLink()
    }

    function createImage(): Image {
        return new MarkdownImage()
    }

    function createTable(): Table {
        return new MarkdownTable()
    }

    function getFileExtension(): String {
        return ".md"
    }
}

class PdfDocumentFactory implements DocumentFactory {
    function createHeading(): Heading {
        return new PdfHeading()
    }

    function createParagraph(): Paragraph {
        return new PdfParagraph()
    }

    function createLink(): Link {
        return new PdfLink()
    }

    function createImage(): Image {
        return new PdfImage()
    }

    function createTable(): Table {
        return new PdfTable()
    }

    function getFileExtension(): String {
        return ".pdf"
    }
}

// ============================================================
// CLIENT CODE
// ============================================================

class DocumentGenerator {
    private factory: DocumentFactory
    private elements: List<DocumentElement> = []

    constructor(factory: DocumentFactory) {
        this.factory = factory
    }

    function addTitle(text: String) {
        heading = factory.createHeading()
        heading.setLevel(1)
        heading.setText(text)
        elements.add(heading)
    }

    function addSection(title: String, content: String) {
        heading = factory.createHeading()
        heading.setLevel(2)
        heading.setText(title)
        elements.add(heading)

        paragraph = factory.createParagraph()
        paragraph.setText(content)
        elements.add(paragraph)
    }

    function addDataTable(headers: List<String>, data: List<List<String>>) {
        table = factory.createTable()
        table.setHeaders(headers)
        for (row in data) {
            table.addRow(row)
        }
        elements.add(table)
    }

    function render(): String {
        output = ""
        for (element in elements) {
            output += element.render() + "\n"
        }
        return output
    }

    function getFileExtension(): String {
        return factory.getFileExtension()
    }
}

// ============================================================
// FACTORY PROVIDER
// ============================================================

class DocumentFactoryProvider {
    private static factories: Map<String, DocumentFactory> = {
        "html": new HtmlDocumentFactory(),
        "markdown": new MarkdownDocumentFactory(),
        "md": new MarkdownDocumentFactory(),
        "pdf": new PdfDocumentFactory()
    }

    static function getFactory(format: String): DocumentFactory {
        factory = factories.get(format.toLowerCase())
        if (factory == null) {
            throw new UnsupportedFormatException(
                "Unknown document format: " + format +
                ". Supported: " + factories.keys().join(", ")
            )
        }
        return factory
    }
}

// ============================================================
// USAGE EXAMPLE
// ============================================================

function main() {
    // Determine format from configuration or user input
    outputFormat = Configuration.get("document.format", "html")

    // Get the appropriate factory
    factory = DocumentFactoryProvider.getFactory(outputFormat)

    // Create document generator with the factory
    generator = new DocumentGenerator(factory)

    // Build the document (client code is identical regardless of format)
    generator.addTitle("Quarterly Sales Report")
    generator.addSection("Overview",
        "This report summarizes sales performance for Q3 2024.")
    generator.addDataTable(
        ["Region", "Sales", "Growth"],
        [
            ["North", "$1.2M", "+15%"],
            ["South", "$890K", "+8%"],
            ["East", "$1.5M", "+22%"],
            ["West", "$950K", "+12%"]
        ]
    )

    // Render and save
    content = generator.render()
    filename = "sales-report" + generator.getFileExtension()
    File.write(filename, content)

    print("Document saved as: " + filename)
}
```

## Known Uses

- **Java AWT**: The `java.awt.Toolkit` class is an abstract factory that creates platform-specific GUI components. Subclasses like `sun.awt.windows.WToolkit` create Windows-specific components.

- **Qt Framework**: QStyleFactory creates platform-specific styles (Windows, macOS, Fusion) that produce consistent widget appearances.

- **Database Drivers**: ADO.NET's `DbProviderFactory` creates database-specific connection, command, and data adapter objects (SqlClientFactory, OracleClientFactory, etc.).

- **Document Parsers**: Libraries like Apache POI use factories to create format-specific document components (XSSF for .xlsx, HSSF for .xls).

- **Game Engines**: Unity and Unreal use abstract factories for cross-platform input handling, rendering backends, and audio systems.

- **Dependency Injection Containers**: Frameworks like Spring and Symfony use abstract factories to create service objects with their dependencies.

- **Logging Frameworks**: SLF4J uses `ILoggerFactory` to create loggers from different logging implementations (Logback, Log4j, JDK logging).

- **Payment Processing**: Payment gateways often use abstract factories to create processor-specific transaction, refund, and subscription objects.

## Related Patterns

- **Factory Method**: Abstract Factory is often implemented using Factory Methods. Each method in the Abstract Factory interface is a Factory Method that creates one type of product.

- **Prototype**: Abstract Factory can be implemented using Prototype when the system needs to be independent of how products are created. Instead of creating products via "new", the factory clones prototypes.

- **Singleton**: Concrete factories are often Singletons because an application typically needs only one instance of a concrete factory per product family.

- **Builder**: Abstract Factory focuses on creating families of products, while Builder focuses on constructing complex objects step by step. They can be combined: a Builder might use an Abstract Factory to create its components.

- **Bridge**: Abstract Factory can be used with Bridge to create platform-specific implementations of abstractions.

## When NOT to Use

- **Single product type**: If you only need to create one type of product, use Factory Method instead. Abstract Factory adds unnecessary complexity when product families aren't needed.

- **Unlikely to change product families**: If your application will only ever use one "family" of products (e.g., only HTML output, never PDF or Markdown), the abstraction overhead isn't justified.

- **Products don't need to work together**: If the products you're creating don't have compatibility constraints or don't need to be used together, Abstract Factory's family concept adds no value.

- **Small applications**: For simple applications with few classes, the pattern introduces too many interfaces and classes. Start with simple direct instantiation and refactor to Abstract Factory when the need arises.

- **Frequently changing product interfaces**: If the AbstractProduct interfaces change often, you'll need to update all concrete product classes across all factories. This creates a maintenance burden.

- **Performance-critical code**: The indirection through interfaces and factory methods adds slight overhead. In extremely performance-sensitive loops, direct instantiation may be preferable.

**Simpler alternatives**:
- **Direct instantiation**: For applications with stable requirements and a single product family
- **Factory Method**: When you only need to create one type of product
- **Simple Factory (not a GoF pattern)**: A single factory class with a method that creates objects based on parameters—less flexible but simpler
- **Dependency Injection**: Modern DI containers often eliminate the need for hand-written factories

**Signs you've over-engineered**:
- You have only one concrete factory
- Your factories have only one creation method
- You're never actually swapping product families at runtime
- The product classes are trivial and don't benefit from abstraction

---

*Based on concepts from "Design Patterns: Elements of Reusable Object-Oriented Software" by Gamma, Helm, Johnson, and Vlissides (Gang of Four), 1994.*
