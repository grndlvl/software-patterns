# Visitor

## Intent

Represent an operation to be performed on the elements of an object structure. Visitor lets you define a new operation without changing the classes of the elements on which it operates.

## Also Known As

- Double Dispatch

## Motivation

Consider a compiler that represents programs as abstract syntax trees (ASTs). The AST contains different node types: assignments, variable references, arithmetic operations, function calls, and so on. The compiler needs to perform many operations on this tree: type checking, optimization, code generation, pretty printing, and static analysis. Each operation needs to treat each node type differently.

The naive approach adds methods for each operation to each node class. The AssignmentNode gets typeCheck(), optimize(), generateCode(), and prettyPrint() methods, as does VariableNode, AdditionNode, and every other node type. This scatters each operation across all node classes and requires modifying every node class to add a new operation.

The Visitor pattern solves this by moving each operation into a separate visitor class. A TypeCheckVisitor contains all type-checking logic in one place, with a method for each node type. An OptimizationVisitor contains all optimization logic, and so forth. Nodes accept visitors and call back with their specific type, enabling the visitor to execute the appropriate method.

This design inverts the matrix. Instead of having N operations each scattered across M element classes, you have N visitor classes each containing M methods. Adding a new operation means adding a new visitor class—no node classes change. The trade-off is that adding a new node type requires updating all visitors.

## Applicability

Use the Visitor pattern when:

- An object structure contains many classes of objects with differing interfaces, and you want to perform operations on these objects that depend on their concrete classes.
- Many distinct and unrelated operations need to be performed on objects in an object structure, and you want to avoid "polluting" their classes with these operations. Visitor lets you keep related operations together.
- The classes defining the object structure rarely change, but you often want to define new operations over the structure. Changing the object structure classes requires redefining the interface to all visitors.
- You need to perform operations across a heterogeneous collection of objects.

Common applications include:
- Compiler AST operations (type checking, code generation, optimization)
- Document object model traversal
- File system operations (calculate size, find files, generate reports)
- GUI element rendering to different formats
- Serialization to multiple formats
- Calculating metrics on code structures
- Report generation from complex data structures

## Structure

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│    ┌───────────────────────────────────────────────────────────────────┐   │
│    │                    <<interface>> Visitor                          │   │
│    ├───────────────────────────────────────────────────────────────────┤   │
│    │ + visitConcreteElementA(element: ConcreteElementA): void          │   │
│    │ + visitConcreteElementB(element: ConcreteElementB): void          │   │
│    │ + visitConcreteElementC(element: ConcreteElementC): void          │   │
│    └───────────────────────────────────────────────────────────────────┘   │
│                                    △                                        │
│                                    │                                        │
│              ┌─────────────────────┴─────────────────────┐                 │
│              │                                           │                 │
│    ┌─────────┴─────────────┐              ┌─────────────┴─────────────┐   │
│    │  ConcreteVisitor1     │              │    ConcreteVisitor2       │   │
│    ├───────────────────────┤              ├───────────────────────────┤   │
│    │ + visitConcreteElemA()│              │ + visitConcreteElementA() │   │
│    │ + visitConcreteElemB()│              │ + visitConcreteElementB() │   │
│    │ + visitConcreteElemC()│              │ + visitConcreteElementC() │   │
│    └───────────────────────┘              └───────────────────────────┘   │
│                                                                             │
│    ┌───────────────────────────────────────────────────────────────────┐   │
│    │                    <<interface>> Element                          │   │
│    ├───────────────────────────────────────────────────────────────────┤   │
│    │ + accept(visitor: Visitor): void                                  │   │
│    └───────────────────────────────────────────────────────────────────┘   │
│                                    △                                        │
│                                    │                                        │
│         ┌──────────────────────────┼──────────────────────────┐            │
│         │                          │                          │            │
│    ┌────┴────────────┐    ┌───────┴───────┐    ┌─────────────┴────┐       │
│    │ConcreteElementA │    │ConcreteElementB│    │ ConcreteElementC │       │
│    ├─────────────────┤    ├───────────────┤    ├──────────────────┤       │
│    │ + accept(v)     │    │ + accept(v)   │    │ + accept(v)      │       │
│    │   v.visitA(this)│    │   v.visitB()  │    │   v.visitC(this) │       │
│    │ + operationA()  │    │ + operationB()│    │ + operationC()   │       │
│    └─────────────────┘    └───────────────┘    └──────────────────┘       │
│                                                                             │
│   Double Dispatch Mechanism:                                               │
│                                                                             │
│   ┌────────┐        ┌─────────┐        ┌─────────┐                        │
│   │ Client │        │ Element │        │ Visitor │                        │
│   └───┬────┘        └────┬────┘        └────┬────┘                        │
│       │                  │                  │                              │
│       │ accept(visitor)  │                  │                              │
│       │─────────────────>│                  │      1st dispatch:          │
│       │                  │                  │      element.accept()       │
│       │                  │ visitXxx(this)   │                              │
│       │                  │─────────────────>│      2nd dispatch:          │
│       │                  │                  │      visitor.visitXxx()     │
│       │                  │                  │                              │
│       │                  │  Access element  │                              │
│       │                  │<─────────────────│      Visitor accesses       │
│       │                  │    data/methods  │      element's interface    │
│       │                  │                  │                              │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Participants

- **Visitor**: Declares a visit operation for each class of ConcreteElement in the object structure. The operation's name and signature identifies the class that sends the visit request to the visitor.

- **ConcreteVisitor**: Implements each operation declared by Visitor. Each operation implements a fragment of the algorithm defined for the corresponding class of object in the structure. ConcreteVisitor provides the context for the algorithm and stores its local state.

- **Element**: Defines an accept operation that takes a visitor as an argument.

- **ConcreteElement**: Implements the accept operation that takes a visitor as an argument. The implementation calls the appropriate visitor method for this element type.

- **ObjectStructure**: Can enumerate its elements. May provide a high-level interface to allow the visitor to visit its elements. May either be a Composite or a collection such as a list or set.

## Collaborations

1. A client creates a ConcreteVisitor object and traverses the object structure, visiting each element with the visitor.

2. When an element is visited, it calls the visitor operation that corresponds to its class. The element supplies itself as an argument to this operation.

3. The visitor can then access the element's interface to perform its operation.

4. This is called "double dispatch" because the operation that gets executed depends on both the type of visitor and the type of element.

## Consequences

### Benefits

1. **Adding new operations is easy**: You can define a new operation over an object structure simply by adding a new visitor. In contrast, spreading functionality across many classes requires changing each class.

2. **Gathering related operations**: A visitor gathers related operations and separates unrelated ones. Related behavior isn't spread over the classes defining the object structure; it's localized in a visitor. Unrelated sets of behavior are partitioned in their own visitor subclasses.

3. **Visiting across class hierarchies**: An iterator can visit objects of different types. Visitor doesn't have this restriction. You can add visit operations to classes that don't share a common parent class.

4. **Accumulating state**: Visitors can accumulate state as they visit each element. Without a visitor, this state would have to be passed as extra arguments to the operations that perform the traversal, or stored in global variables.

### Liabilities

1. **Adding new ConcreteElement classes is hard**: Each new element requires a new abstract operation on Visitor and a corresponding implementation in every ConcreteVisitor. Consider whether you're likely to add elements or operations more often.

2. **Breaking encapsulation**: Visitor's approach assumes that the ConcreteElement interface is powerful enough to let visitors do their job. This may force you to provide public operations that access an element's internal state, compromising encapsulation.

3. **Complexity**: The double dispatch mechanism can be confusing for developers unfamiliar with the pattern.

## Implementation

### Implementation Considerations

1. **Double dispatch**: The visitor pattern uses double dispatch. The operation that gets executed depends on both the kind of request and the type of receiver. `accept` is a double-dispatch operation—its meaning depends on two types: the visitor's and the element's.

2. **Who is responsible for traversing?**: The object structure could do it, the visitor could do it, or a separate iterator could. Object structure traversal is common when the structure has a complex organization.

3. **Visitor accumulating results**: Visitors can accumulate results during traversal. Without a visitor, these results would need to be passed as parameters or stored globally.

### Pseudocode: Visitor Infrastructure

```
// Visitor interface - declares visit method for each element type
interface DocumentVisitor {
    method visitParagraph(paragraph: Paragraph): void
    method visitHeading(heading: Heading): void
    method visitImage(image: Image): void
    method visitTable(table: Table): void
    method visitList(list: ListElement): void
    method visitLink(link: Link): void
    method visitCodeBlock(codeBlock: CodeBlock): void
}

// Element interface
interface DocumentElement {
    method accept(visitor: DocumentVisitor): void
    method getChildren(): list<DocumentElement>
}

// Base class for elements that can contain children
abstract class CompositeElement implements DocumentElement {
    protected children: list<DocumentElement> = []

    method addChild(child: DocumentElement): void {
        this.children.add(child)
    }

    method getChildren(): list<DocumentElement> {
        return this.children.copy()
    }
}
```

### Pseudocode: Concrete Elements

```
// Paragraph element
class Paragraph extends CompositeElement {
    public text: string
    public alignment: string = "left"
    public indent: int = 0

    constructor(text: string) {
        this.text = text
    }

    method accept(visitor: DocumentVisitor): void {
        visitor.visitParagraph(this)
    }
}

// Heading element
class Heading implements DocumentElement {
    public level: int  // 1-6
    public text: string

    constructor(level: int, text: string) {
        this.level = level
        this.text = text
    }

    method accept(visitor: DocumentVisitor): void {
        visitor.visitHeading(this)
    }

    method getChildren(): list<DocumentElement> {
        return []
    }
}

// Image element
class Image implements DocumentElement {
    public src: string
    public alt: string
    public width: int
    public height: int
    public caption: string

    constructor(src: string, alt: string) {
        this.src = src
        this.alt = alt
    }

    method accept(visitor: DocumentVisitor): void {
        visitor.visitImage(this)
    }

    method getChildren(): list<DocumentElement> {
        return []
    }
}

// Table element
class Table extends CompositeElement {
    public headers: list<string>
    public rows: list<list<string>>

    constructor(headers: list<string>) {
        this.headers = headers
        this.rows = []
    }

    method addRow(row: list<string>): void {
        this.rows.add(row)
    }

    method accept(visitor: DocumentVisitor): void {
        visitor.visitTable(this)
    }
}

// List element
class ListElement extends CompositeElement {
    public ordered: boolean
    public items: list<string>

    constructor(ordered: boolean = false) {
        this.ordered = ordered
        this.items = []
    }

    method addItem(item: string): void {
        this.items.add(item)
    }

    method accept(visitor: DocumentVisitor): void {
        visitor.visitList(this)
    }
}

// Link element
class Link implements DocumentElement {
    public href: string
    public text: string
    public title: string

    constructor(href: string, text: string) {
        this.href = href
        this.text = text
    }

    method accept(visitor: DocumentVisitor): void {
        visitor.visitLink(this)
    }

    method getChildren(): list<DocumentElement> {
        return []
    }
}

// Code block element
class CodeBlock implements DocumentElement {
    public code: string
    public language: string

    constructor(code: string, language: string = "") {
        this.code = code
        this.language = language
    }

    method accept(visitor: DocumentVisitor): void {
        visitor.visitCodeBlock(this)
    }

    method getChildren(): list<DocumentElement> {
        return []
    }
}
```

### Pseudocode: Concrete Visitors

```
// HTML Export Visitor
class HTMLExportVisitor implements DocumentVisitor {
    private output: StringBuilder = new StringBuilder()
    private indentLevel: int = 0

    method getResult(): string {
        return this.output.toString()
    }

    method visitParagraph(paragraph: Paragraph): void {
        style = ""
        if paragraph.alignment != "left" {
            style = " style=\"text-align: " + paragraph.alignment + ";\""
        }

        this.writeLine("<p" + style + ">")
        this.indentLevel++
        this.writeLine(this.escapeHtml(paragraph.text))

        // Visit children (inline elements)
        for child in paragraph.getChildren() {
            child.accept(this)
        }

        this.indentLevel--
        this.writeLine("</p>")
    }

    method visitHeading(heading: Heading): void {
        tag = "h" + heading.level
        this.writeLine("<" + tag + ">" + this.escapeHtml(heading.text) + "</" + tag + ">")
    }

    method visitImage(image: Image): void {
        attrs = "src=\"" + image.src + "\" alt=\"" + this.escapeHtml(image.alt) + "\""
        if image.width > 0 {
            attrs = attrs + " width=\"" + image.width + "\""
        }
        if image.height > 0 {
            attrs = attrs + " height=\"" + image.height + "\""
        }

        if image.caption.isNotEmpty() {
            this.writeLine("<figure>")
            this.indentLevel++
            this.writeLine("<img " + attrs + ">")
            this.writeLine("<figcaption>" + this.escapeHtml(image.caption) + "</figcaption>")
            this.indentLevel--
            this.writeLine("</figure>")
        } else {
            this.writeLine("<img " + attrs + ">")
        }
    }

    method visitTable(table: Table): void {
        this.writeLine("<table>")
        this.indentLevel++

        // Headers
        this.writeLine("<thead>")
        this.indentLevel++
        this.writeLine("<tr>")
        this.indentLevel++
        for header in table.headers {
            this.writeLine("<th>" + this.escapeHtml(header) + "</th>")
        }
        this.indentLevel--
        this.writeLine("</tr>")
        this.indentLevel--
        this.writeLine("</thead>")

        // Body
        this.writeLine("<tbody>")
        this.indentLevel++
        for row in table.rows {
            this.writeLine("<tr>")
            this.indentLevel++
            for cell in row {
                this.writeLine("<td>" + this.escapeHtml(cell) + "</td>")
            }
            this.indentLevel--
            this.writeLine("</tr>")
        }
        this.indentLevel--
        this.writeLine("</tbody>")

        this.indentLevel--
        this.writeLine("</table>")
    }

    method visitList(list: ListElement): void {
        tag = list.ordered ? "ol" : "ul"
        this.writeLine("<" + tag + ">")
        this.indentLevel++
        for item in list.items {
            this.writeLine("<li>" + this.escapeHtml(item) + "</li>")
        }
        this.indentLevel--
        this.writeLine("</" + tag + ">")
    }

    method visitLink(link: Link): void {
        title = link.title.isNotEmpty() ? " title=\"" + this.escapeHtml(link.title) + "\"" : ""
        this.write("<a href=\"" + link.href + "\"" + title + ">" +
                   this.escapeHtml(link.text) + "</a>")
    }

    method visitCodeBlock(codeBlock: CodeBlock): void {
        langClass = codeBlock.language.isNotEmpty() ?
                    " class=\"language-" + codeBlock.language + "\"" : ""
        this.writeLine("<pre><code" + langClass + ">")
        this.write(this.escapeHtml(codeBlock.code))
        this.writeLine("</code></pre>")
    }

    private method writeLine(text: string): void {
        indent = "  ".repeat(this.indentLevel)
        this.output.append(indent + text + "\n")
    }

    private method write(text: string): void {
        this.output.append(text)
    }

    private method escapeHtml(text: string): string {
        return text
            .replace("&", "&amp;")
            .replace("<", "&lt;")
            .replace(">", "&gt;")
            .replace("\"", "&quot;")
    }
}

// Markdown Export Visitor
class MarkdownExportVisitor implements DocumentVisitor {
    private output: StringBuilder = new StringBuilder()

    method getResult(): string {
        return this.output.toString()
    }

    method visitParagraph(paragraph: Paragraph): void {
        this.output.append(paragraph.text)
        for child in paragraph.getChildren() {
            child.accept(this)
        }
        this.output.append("\n\n")
    }

    method visitHeading(heading: Heading): void {
        prefix = "#".repeat(heading.level)
        this.output.append(prefix + " " + heading.text + "\n\n")
    }

    method visitImage(image: Image): void {
        alt = image.alt.isNotEmpty() ? image.alt : "image"
        this.output.append("![" + alt + "](" + image.src + ")")
        if image.caption.isNotEmpty() {
            this.output.append("\n*" + image.caption + "*")
        }
        this.output.append("\n\n")
    }

    method visitTable(table: Table): void {
        // Headers
        this.output.append("| " + join(table.headers, " | ") + " |\n")
        // Separator
        this.output.append("| " + join(table.headers.map(h => "---"), " | ") + " |\n")
        // Rows
        for row in table.rows {
            this.output.append("| " + join(row, " | ") + " |\n")
        }
        this.output.append("\n")
    }

    method visitList(list: ListElement): void {
        for i, item in list.items.enumerate() {
            prefix = list.ordered ? (i + 1) + ". " : "- "
            this.output.append(prefix + item + "\n")
        }
        this.output.append("\n")
    }

    method visitLink(link: Link): void {
        this.output.append("[" + link.text + "](" + link.href + ")")
    }

    method visitCodeBlock(codeBlock: CodeBlock): void {
        this.output.append("```" + codeBlock.language + "\n")
        this.output.append(codeBlock.code + "\n")
        this.output.append("```\n\n")
    }
}

// Statistics Visitor - accumulates data about document
class DocumentStatisticsVisitor implements DocumentVisitor {
    private wordCount: int = 0
    private charCount: int = 0
    private imageCount: int = 0
    private linkCount: int = 0
    private tableCount: int = 0
    private headings: map<int, int> = {}  // level -> count
    private codeBlocks: map<string, int> = {}  // language -> count

    method getStatistics(): DocumentStatistics {
        return new DocumentStatistics(
            wordCount: this.wordCount,
            charCount: this.charCount,
            imageCount: this.imageCount,
            linkCount: this.linkCount,
            tableCount: this.tableCount,
            headingsByLevel: this.headings,
            codeBlocksByLanguage: this.codeBlocks
        )
    }

    method visitParagraph(paragraph: Paragraph): void {
        this.countText(paragraph.text)
        for child in paragraph.getChildren() {
            child.accept(this)
        }
    }

    method visitHeading(heading: Heading): void {
        this.countText(heading.text)
        currentCount = this.headings.getOrDefault(heading.level, 0)
        this.headings.set(heading.level, currentCount + 1)
    }

    method visitImage(image: Image): void {
        this.imageCount++
        if image.caption.isNotEmpty() {
            this.countText(image.caption)
        }
    }

    method visitTable(table: Table): void {
        this.tableCount++
        for header in table.headers {
            this.countText(header)
        }
        for row in table.rows {
            for cell in row {
                this.countText(cell)
            }
        }
    }

    method visitList(list: ListElement): void {
        for item in list.items {
            this.countText(item)
        }
    }

    method visitLink(link: Link): void {
        this.linkCount++
        this.countText(link.text)
    }

    method visitCodeBlock(codeBlock: CodeBlock): void {
        lang = codeBlock.language.isNotEmpty() ? codeBlock.language : "unknown"
        currentCount = this.codeBlocks.getOrDefault(lang, 0)
        this.codeBlocks.set(lang, currentCount + 1)
        // Don't count code as words
        this.charCount = this.charCount + codeBlock.code.length
    }

    private method countText(text: string): void {
        this.charCount = this.charCount + text.length
        words = text.split("\\s+").filter(w => w.isNotEmpty())
        this.wordCount = this.wordCount + words.size()
    }
}

// Validation Visitor - checks document for issues
class DocumentValidationVisitor implements DocumentVisitor {
    private errors: list<ValidationError> = []
    private warnings: list<ValidationWarning> = []

    method getErrors(): list<ValidationError> {
        return this.errors.copy()
    }

    method getWarnings(): list<ValidationWarning> {
        return this.warnings.copy()
    }

    method isValid(): boolean {
        return this.errors.isEmpty()
    }

    method visitParagraph(paragraph: Paragraph): void {
        if paragraph.text.trim().isEmpty() and paragraph.getChildren().isEmpty() {
            this.warnings.add(new ValidationWarning("Empty paragraph found"))
        }
        for child in paragraph.getChildren() {
            child.accept(this)
        }
    }

    method visitHeading(heading: Heading): void {
        if heading.level < 1 or heading.level > 6 {
            this.errors.add(new ValidationError(
                "Invalid heading level: " + heading.level
            ))
        }
        if heading.text.trim().isEmpty() {
            this.errors.add(new ValidationError("Heading has no text"))
        }
    }

    method visitImage(image: Image): void {
        if image.src.isEmpty() {
            this.errors.add(new ValidationError("Image has no source URL"))
        }
        if image.alt.isEmpty() {
            this.warnings.add(new ValidationWarning(
                "Image missing alt text: " + image.src
            ))
        }
        if not this.isValidUrl(image.src) {
            this.warnings.add(new ValidationWarning(
                "Potentially invalid image URL: " + image.src
            ))
        }
    }

    method visitTable(table: Table): void {
        if table.headers.isEmpty() {
            this.warnings.add(new ValidationWarning("Table has no headers"))
        }
        headerCount = table.headers.size()
        for i, row in table.rows.enumerate() {
            if row.size() != headerCount {
                this.errors.add(new ValidationError(
                    "Table row " + (i + 1) + " has " + row.size() +
                    " cells, expected " + headerCount
                ))
            }
        }
    }

    method visitList(list: ListElement): void {
        if list.items.isEmpty() {
            this.warnings.add(new ValidationWarning("Empty list found"))
        }
    }

    method visitLink(link: Link): void {
        if link.href.isEmpty() {
            this.errors.add(new ValidationError("Link has no href"))
        }
        if link.text.isEmpty() {
            this.warnings.add(new ValidationWarning(
                "Link has no text: " + link.href
            ))
        }
        if not this.isValidUrl(link.href) and not link.href.startsWith("#") {
            this.warnings.add(new ValidationWarning(
                "Potentially invalid link URL: " + link.href
            ))
        }
    }

    method visitCodeBlock(codeBlock: CodeBlock): void {
        if codeBlock.code.trim().isEmpty() {
            this.warnings.add(new ValidationWarning("Empty code block found"))
        }
        if codeBlock.language.isEmpty() {
            this.warnings.add(new ValidationWarning(
                "Code block has no language specified"
            ))
        }
    }

    private method isValidUrl(url: string): boolean {
        return url.startsWith("http://") or
               url.startsWith("https://") or
               url.startsWith("/") or
               url.startsWith("./")
    }
}
```

## Example

A complete example implementing visitors for a file system:

```
// File system elements
interface FileSystemElement {
    method accept(visitor: FileSystemVisitor): void
    method getName(): string
    method getPath(): string
}

class File implements FileSystemElement {
    public name: string
    public path: string
    public size: long
    public extension: string
    public createdAt: datetime
    public modifiedAt: datetime
    public permissions: string

    constructor(path: string) {
        this.path = path
        this.name = extractFileName(path)
        this.extension = extractExtension(path)
        // Load metadata from filesystem
        metadata = getFileMetadata(path)
        this.size = metadata.size
        this.createdAt = metadata.createdAt
        this.modifiedAt = metadata.modifiedAt
        this.permissions = metadata.permissions
    }

    method accept(visitor: FileSystemVisitor): void {
        visitor.visitFile(this)
    }

    method getName(): string {
        return this.name
    }

    method getPath(): string {
        return this.path
    }
}

class Directory implements FileSystemElement {
    public name: string
    public path: string
    public createdAt: datetime
    public modifiedAt: datetime
    private children: list<FileSystemElement> = []

    constructor(path: string) {
        this.path = path
        this.name = extractFileName(path)
        metadata = getFileMetadata(path)
        this.createdAt = metadata.createdAt
        this.modifiedAt = metadata.modifiedAt
    }

    method addChild(element: FileSystemElement): void {
        this.children.add(element)
    }

    method getChildren(): list<FileSystemElement> {
        return this.children.copy()
    }

    method accept(visitor: FileSystemVisitor): void {
        visitor.visitDirectory(this)
    }

    method getName(): string {
        return this.name
    }

    method getPath(): string {
        return this.path
    }
}

class SymbolicLink implements FileSystemElement {
    public name: string
    public path: string
    public target: string
    public isBroken: boolean

    constructor(path: string) {
        this.path = path
        this.name = extractFileName(path)
        this.target = readLink(path)
        this.isBroken = not fileExists(this.target)
    }

    method accept(visitor: FileSystemVisitor): void {
        visitor.visitSymlink(this)
    }

    method getName(): string {
        return this.name
    }

    method getPath(): string {
        return this.path
    }
}

// Visitor interface
interface FileSystemVisitor {
    method visitFile(file: File): void
    method visitDirectory(directory: Directory): void
    method visitSymlink(symlink: SymbolicLink): void
}

// Size calculation visitor
class SizeCalculatorVisitor implements FileSystemVisitor {
    private totalSize: long = 0
    private fileCount: int = 0
    private directoryCount: int = 0

    method getTotalSize(): long {
        return this.totalSize
    }

    method getFileCount(): int {
        return this.fileCount
    }

    method getDirectoryCount(): int {
        return this.directoryCount
    }

    method getFormattedSize(): string {
        return formatBytes(this.totalSize)
    }

    method visitFile(file: File): void {
        this.totalSize = this.totalSize + file.size
        this.fileCount++
    }

    method visitDirectory(directory: Directory): void {
        this.directoryCount++
        // Visit children to accumulate sizes
        for child in directory.getChildren() {
            child.accept(this)
        }
    }

    method visitSymlink(symlink: SymbolicLink): void {
        // Symlinks don't contribute to size (or follow the link)
    }
}

// Find files by pattern visitor
class FileFinderVisitor implements FileSystemVisitor {
    private pattern: Regex
    private matches: list<File> = []
    private includeDirectories: boolean

    constructor(pattern: string, includeDirectories: boolean = false) {
        this.pattern = new Regex(pattern)
        this.includeDirectories = includeDirectories
    }

    method getMatches(): list<File> {
        return this.matches.copy()
    }

    method visitFile(file: File): void {
        if this.pattern.matches(file.name) {
            this.matches.add(file)
        }
    }

    method visitDirectory(directory: Directory): void {
        if this.includeDirectories and this.pattern.matches(directory.name) {
            // Add directory to matches (would need to modify return type)
        }
        // Continue searching in children
        for child in directory.getChildren() {
            child.accept(this)
        }
    }

    method visitSymlink(symlink: SymbolicLink): void {
        // Don't follow symlinks in search
    }
}

// Duplicate file finder visitor
class DuplicateFinderVisitor implements FileSystemVisitor {
    private filesBySize: map<long, list<File>> = {}
    private duplicates: list<list<File>> = []
    private minSize: long

    constructor(minSize: long = 1) {
        this.minSize = minSize
    }

    method findDuplicates(): list<list<File>> {
        this.duplicates = []

        // Check files with same size
        for size, files in this.filesBySize {
            if files.size() > 1 {
                // Group by hash
                filesByHash = new map<string, list<File>>()
                for file in files {
                    hash = calculateFileHash(file.path)
                    if not filesByHash.containsKey(hash) {
                        filesByHash.set(hash, [])
                    }
                    filesByHash.get(hash).add(file)
                }

                // Add groups with duplicates
                for hash, hashFiles in filesByHash {
                    if hashFiles.size() > 1 {
                        this.duplicates.add(hashFiles)
                    }
                }
            }
        }

        return this.duplicates
    }

    method visitFile(file: File): void {
        if file.size < this.minSize {
            return
        }

        if not this.filesBySize.containsKey(file.size) {
            this.filesBySize.set(file.size, [])
        }
        this.filesBySize.get(file.size).add(file)
    }

    method visitDirectory(directory: Directory): void {
        for child in directory.getChildren() {
            child.accept(this)
        }
    }

    method visitSymlink(symlink: SymbolicLink): void {
        // Skip symlinks
    }
}

// Export to tree format visitor
class TreeExportVisitor implements FileSystemVisitor {
    private output: StringBuilder = new StringBuilder()
    private depth: int = 0
    private isLast: list<boolean> = []

    method getResult(): string {
        return this.output.toString()
    }

    method visitFile(file: File): void {
        this.printEntry(file.name, false)
    }

    method visitDirectory(directory: Directory): void {
        this.printEntry(directory.name + "/", true)

        children = directory.getChildren()
        this.depth++

        for i, child in children.enumerate() {
            isLastChild = (i == children.size() - 1)
            this.isLast.add(isLastChild)
            child.accept(this)
            this.isLast.removeLast()
        }

        this.depth--
    }

    method visitSymlink(symlink: SymbolicLink): void {
        suffix = " -> " + symlink.target
        if symlink.isBroken {
            suffix = suffix + " [broken]"
        }
        this.printEntry(symlink.name + suffix, false)
    }

    private method printEntry(name: string, isDir: boolean): void {
        prefix = this.buildPrefix()
        this.output.append(prefix + name + "\n")
    }

    private method buildPrefix(): string {
        if this.depth == 0 {
            return ""
        }

        prefix = ""
        for i = 0 to this.depth - 2 {
            if i < this.isLast.size() and not this.isLast.get(i) {
                prefix = prefix + "|   "
            } else {
                prefix = prefix + "    "
            }
        }

        if this.isLast.isNotEmpty() and this.isLast.last() {
            prefix = prefix + "`-- "
        } else {
            prefix = prefix + "|-- "
        }

        return prefix
    }
}

// Permission audit visitor
class PermissionAuditVisitor implements FileSystemVisitor {
    private worldWritable: list<FileSystemElement> = []
    private worldReadable: list<FileSystemElement> = []
    private noOwnerRead: list<FileSystemElement> = []

    method getWorldWritableFiles(): list<FileSystemElement> {
        return this.worldWritable.copy()
    }

    method getSecurityReport(): SecurityReport {
        return new SecurityReport(
            worldWritable: this.worldWritable,
            worldReadable: this.worldReadable,
            noOwnerRead: this.noOwnerRead
        )
    }

    method visitFile(file: File): void {
        this.auditPermissions(file, file.permissions)
    }

    method visitDirectory(directory: Directory): void {
        // Audit directory permissions (would need to add permissions field)
        for child in directory.getChildren() {
            child.accept(this)
        }
    }

    method visitSymlink(symlink: SymbolicLink): void {
        // Symlink permissions don't matter much
    }

    private method auditPermissions(element: FileSystemElement, perms: string): void {
        // Assuming Unix-style permissions like "rwxr-xr-x"
        if perms.length >= 9 {
            // World writable (position 8 is 'w' for world write)
            if perms[8] == 'w' {
                this.worldWritable.add(element)
            }
            // World readable
            if perms[7] == 'r' {
                this.worldReadable.add(element)
            }
            // Owner can't read own file
            if perms[0] != 'r' {
                this.noOwnerRead.add(element)
            }
        }
    }
}

// File system traverser that builds the structure
class FileSystemTraverser {
    method traverse(rootPath: string): Directory {
        root = new Directory(rootPath)
        this.populateDirectory(root)
        return root
    }

    private method populateDirectory(directory: Directory): void {
        entries = listDirectory(directory.getPath())

        for entry in entries {
            fullPath = directory.getPath() + "/" + entry

            if isSymlink(fullPath) {
                directory.addChild(new SymbolicLink(fullPath))
            } else if isDirectory(fullPath) {
                subDir = new Directory(fullPath)
                this.populateDirectory(subDir)
                directory.addChild(subDir)
            } else {
                directory.addChild(new File(fullPath))
            }
        }
    }
}

// Usage example
function main() {
    // Build file system structure
    traverser = new FileSystemTraverser()
    root = traverser.traverse("/home/user/projects")

    // Calculate total size
    sizeVisitor = new SizeCalculatorVisitor()
    root.accept(sizeVisitor)
    print("Total size: " + sizeVisitor.getFormattedSize())
    print("Files: " + sizeVisitor.getFileCount())
    print("Directories: " + sizeVisitor.getDirectoryCount())

    // Find all JavaScript files
    jsFinderVisitor = new FileFinderVisitor(".*\\.js$")
    root.accept(jsFinderVisitor)
    print("\nJavaScript files:")
    for file in jsFinderVisitor.getMatches() {
        print("  " + file.getPath())
    }

    // Find duplicate files
    dupVisitor = new DuplicateFinderVisitor(1024)  // Min 1KB
    root.accept(dupVisitor)
    duplicates = dupVisitor.findDuplicates()
    print("\nDuplicate files:")
    for group in duplicates {
        print("  Duplicates (" + formatBytes(group[0].size) + "):")
        for file in group {
            print("    " + file.getPath())
        }
    }

    // Export tree view
    treeVisitor = new TreeExportVisitor()
    root.accept(treeVisitor)
    print("\nDirectory tree:")
    print(treeVisitor.getResult())

    // Security audit
    auditVisitor = new PermissionAuditVisitor()
    root.accept(auditVisitor)
    report = auditVisitor.getSecurityReport()
    if report.worldWritable.isNotEmpty() {
        print("\nWARNING: World-writable files found:")
        for file in report.worldWritable {
            print("  " + file.getPath())
        }
    }
}
```

## Known Uses

- **Compiler Design**: LLVM, GCC, and most compilers use visitors for AST traversal and transformation.

- **Java Annotation Processing**: `javax.lang.model.element.ElementVisitor` visits different kinds of program elements.

- **DOM Traversal**: `org.w3c.dom.traversal.NodeIterator` and tree walkers use visitor-like patterns.

- **Eclipse JDT**: The Java Development Tools use `ASTVisitor` for analyzing and transforming Java code.

- **ANTLR**: The parser generator creates visitor classes for traversing parse trees.

- **Babel**: The JavaScript transpiler uses visitors to transform AST nodes.

- **ESLint**: Linting rules are implemented as visitors over the AST.

- **Java `FileVisitor`**: `java.nio.file.FileVisitor` visits files in a file tree.

- **Roslyn (.NET Compiler)**: C# and VB.NET compilers use syntax visitors extensively.

## Related Patterns

- **Composite**: Visitor is often applied to structures built with Composite. The composite's accept method traverses children.

- **Iterator**: Both traverse structures. Iterator provides elements sequentially; Visitor performs operations on elements based on their type.

- **Interpreter**: Interpreter can use Visitor to interpret an AST. Each visitor method handles a different expression type.

## When NOT to Use

1. **Frequently changing element classes**: If you add new ConcreteElement classes often, you'll need to update all visitors each time. Visitor works best with stable element hierarchies.

```
// Poor fit: Element types change frequently
// Every new shape requires updating ALL visitors
interface ShapeVisitor {
    visitCircle(c: Circle)
    visitSquare(s: Square)
    visitTriangle(t: Triangle)
    visitPentagon(p: Pentagon)    // Added last month
    visitHexagon(h: Hexagon)      // Added last week
    visitOctagon(o: Octagon)      // Added yesterday
    // ... visitors keep growing
}
```

2. **Simple operations that don't vary by type**: If the operation is the same for all elements, you don't need visitor's type dispatch.

```
// Overkill: Same operation for all types
class PrintNameVisitor {
    visitA(a) { print(a.name) }
    visitB(b) { print(b.name) }
    visitC(c) { print(c.name) }
}

// Just use polymorphism
interface Element {
    getName(): string
}
for element in elements {
    print(element.getName())
}
```

3. **When encapsulation is critical**: Visitors often need access to element internals, potentially breaking encapsulation.

4. **Single operation, many element types**: If you have one operation but many element types, you're creating a lot of infrastructure for little benefit.

5. **When type information is available at runtime**: In dynamically-typed languages or when using reflection, pattern matching or type checks may be simpler than visitor.

```
// In languages with pattern matching, this may be clearer
match element {
    case Circle(r) => pi * r * r
    case Square(s) => s * s
    case Triangle(b, h) => 0.5 * b * h
}
```

6. **Performance-critical code**: The double dispatch mechanism adds overhead compared to direct method calls.

The Visitor pattern excels when you have a stable set of element types and need to perform many different operations on them. It's particularly valuable in compilers, document processors, and other systems with well-defined object models. Avoid it when element types change frequently or when simpler solutions suffice.

---

*Based on concepts from "Design Patterns: Elements of Reusable Object-Oriented Software" by Gamma, Helm, Johnson, and Vlissides (Gang of Four), 1994.*
