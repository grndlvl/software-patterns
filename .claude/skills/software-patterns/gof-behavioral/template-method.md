# Template Method

## Intent

Define the skeleton of an algorithm in an operation, deferring some steps to subclasses. Template Method lets subclasses redefine certain steps of an algorithm without changing the algorithm's structure.

## Also Known As

- Hollywood Principle Implementation ("Don't call us, we'll call you")

## Motivation

Consider a data mining application that analyzes documents from different sources: PDF files, CSV spreadsheets, and HTML web pages. Despite their different formats, the analysis follows the same general pattern: open the document, extract raw data, parse the data into a structured format, analyze the data, and generate a report. The overall algorithm is identical; only the extraction and parsing steps differ by format.

The naive approach duplicates the algorithm in each document handler. The PDF analyzer has its open-extract-parse-analyze-report sequence, and so does the CSV analyzer and HTML analyzer. This duplication is problematic: fixing a bug in the analysis step requires changes in three places, and adding a new step (like validation) means modifying every handler.

The Template Method pattern solves this by defining the algorithm's skeleton in a base class method—the template method. This method calls other methods that represent the algorithm's steps. Some steps have default implementations; others are abstract, forcing subclasses to provide implementations. The overall flow is fixed in the base class; subclasses only customize specific steps.

This inverts the typical control flow. Instead of subclasses calling inherited methods, the inherited template method calls subclass methods. This is the Hollywood Principle: "Don't call us, we'll call you." The base class controls the algorithm's structure; subclasses fill in the details.

## Applicability

Use the Template Method pattern when:

- You want to implement the invariant parts of an algorithm once and leave it up to subclasses to implement the behavior that can vary.
- Common behavior among subclasses should be factored and localized in a common class to avoid code duplication. You first identify the differences in the existing code and then separate the differences into new operations. Finally, you replace the differing code with a template method that calls one of these new operations.
- You want to control subclass extensions. You can define a template method that calls "hook" operations at specific points, thereby permitting extensions only at those points.
- You have several classes with similar algorithms that differ in specific steps.

Common applications include:
- Framework initialization sequences
- Document processing pipelines
- Test fixtures (setUp, test, tearDown)
- Build processes
- Data import/export procedures
- Network protocol implementations
- Game loops

## Structure

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│    ┌───────────────────────────────────────────────────────────────────┐   │
│    │                    AbstractClass                                  │   │
│    ├───────────────────────────────────────────────────────────────────┤   │
│    │ + templateMethod(): void      // Defines algorithm skeleton       │   │
│    │   {                                                               │   │
│    │       step1();                // Concrete - shared by all        │   │
│    │       step2();                // Abstract - subclass provides    │   │
│    │       step3();                // Abstract - subclass provides    │   │
│    │       hook();                 // Hook - optional override        │   │
│    │       step4();                // Concrete - shared by all        │   │
│    │   }                                                               │   │
│    │                                                                   │   │
│    │ # step1(): void               // Concrete operation              │   │
│    │ # step2(): void {abstract}    // Primitive operation (required)  │   │
│    │ # step3(): void {abstract}    // Primitive operation (required)  │   │
│    │ # hook(): void { }            // Hook operation (optional)       │   │
│    │ # step4(): void               // Concrete operation              │   │
│    └───────────────────────────────────────────────────────────────────┘   │
│                                    △                                        │
│                                    │                                        │
│              ┌─────────────────────┴─────────────────────┐                 │
│              │                                           │                 │
│    ┌─────────┴─────────────┐              ┌─────────────┴─────────────┐   │
│    │   ConcreteClassA      │              │     ConcreteClassB        │   │
│    ├───────────────────────┤              ├───────────────────────────┤   │
│    │ # step2(): void       │              │ # step2(): void           │   │
│    │   // Implementation A │              │   // Implementation B     │   │
│    │ # step3(): void       │              │ # step3(): void           │   │
│    │   // Implementation A │              │   // Implementation B     │   │
│    │ # hook(): void        │              │   // Uses default hook    │   │
│    │   // Custom hook      │              │                           │   │
│    └───────────────────────┘              └───────────────────────────┘   │
│                                                                             │
│   Template Method Flow:                                                     │
│                                                                             │
│   ┌─────────┐                                                              │
│   │ Client  │                                                              │
│   └────┬────┘                                                              │
│        │                                                                    │
│        │  templateMethod()                                                  │
│        ▼                                                                    │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                          AbstractClass                              │  │
│   │  ┌──────────────────────────────────────────────────────────────┐  │  │
│   │  │  templateMethod() {                                          │  │  │
│   │  │      step1();        ────────> AbstractClass.step1()         │  │  │
│   │  │      step2();        ────────> ConcreteClass.step2()  ◄──┐  │  │  │
│   │  │      step3();        ────────> ConcreteClass.step3()  ◄──┤  │  │  │
│   │  │      hook();         ────────> ConcreteClass.hook()   ◄──┤  │  │  │
│   │  │      step4();        ────────> AbstractClass.step4()     │  │  │  │
│   │  │  }                                                    Subclass│  │  │
│   │  └──────────────────────────────────────────────────────provides┘  │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Participants

- **AbstractClass**: Defines abstract primitive operations that concrete subclasses define to implement steps of an algorithm. Implements a template method defining the skeleton of an algorithm. The template method calls primitive operations as well as operations defined in AbstractClass or those of other objects.

- **ConcreteClass**: Implements the primitive operations to carry out subclass-specific steps of the algorithm. May also override hook operations.

## Collaborations

1. ConcreteClass relies on AbstractClass to implement the invariant steps of the algorithm.

2. The template method in AbstractClass controls the overall algorithm structure and calls primitive/hook operations that ConcreteClass implements.

3. Clients call the template method, not the individual steps. The template method orchestrates the steps in the correct order.

## Consequences

### Benefits

1. **Code reuse**: Template methods are a fundamental technique for code reuse. They are particularly important in class libraries because they factor out common behavior.

2. **Inverted control structure**: With template methods, the parent class calls the subclass operations, not the other way around. This is often called the "Hollywood Principle."

3. **Enforces algorithm structure**: The template method ensures the algorithm follows a specific structure. Subclasses can't change the order of steps or skip steps.

4. **Provides hooks for extension**: Hook operations provide default behavior that subclasses can extend if necessary. A hook operation often does nothing by default.

5. **Consistent interface**: All concrete implementations share the same interface (the template method), making them interchangeable.

### Liabilities

1. **Limited flexibility**: The algorithm's structure is fixed. If subclasses need radically different structures, Template Method may not be appropriate.

2. **Inheritance coupling**: Subclasses are tightly coupled to the abstract class. Changes to the template method affect all subclasses.

3. **Violation of Liskov Substitution Principle**: If subclasses override methods in ways that break expected behavior, it can violate LSP.

4. **Complexity with many hooks**: Too many hook points can make the template method hard to understand and maintain.

5. **Hard to maintain with deep hierarchies**: If concrete classes introduce their own template methods, the hierarchy becomes complex.

## Implementation

### Implementation Considerations

1. **Minimizing primitive operations**: The more primitive operations a template method defines, the more tedious it is to implement concrete subclasses. It's important to keep the number of primitive operations small.

2. **Naming conventions**: Consider using a naming convention to identify operations that must be overridden (abstract) versus those that may be overridden (hooks). For example, prefix hooks with "do" or "on".

3. **Using access control**: In languages that support it, make primitive operations protected. This prevents clients from calling them directly while allowing subclasses to override them.

4. **Making the template method final**: Consider making the template method final (non-overridable) to prevent subclasses from changing the algorithm structure.

### Pseudocode: Template Method Infrastructure

```
// Abstract class with template method
abstract class DataMiner {
    // The template method defines the algorithm skeleton
    // Made final to prevent subclasses from changing the structure
    final method mine(path: string): AnalysisReport {
        // Step 1: Open the data source
        file = this.openFile(path)

        // Step 2: Extract raw data (abstract - subclass must implement)
        rawData = this.extractData(file)

        // Step 3: Parse the raw data (abstract - subclass must implement)
        parsedData = this.parseData(rawData)

        // Step 4: Hook - optional preprocessing
        processedData = this.preprocessData(parsedData)

        // Step 5: Analyze the data (concrete - shared implementation)
        analysis = this.analyzeData(processedData)

        // Step 6: Hook - optional post-processing
        this.postProcess(analysis)

        // Step 7: Generate report (concrete - shared implementation)
        report = this.generateReport(analysis)

        // Step 8: Close resources
        this.closeFile(file)

        return report
    }

    // Concrete operation - shared by all subclasses
    protected method openFile(path: string): FileHandle {
        log("Opening file: " + path)
        return new FileHandle(path)
    }

    // Abstract primitive operation - must be implemented by subclasses
    protected abstract method extractData(file: FileHandle): bytes

    // Abstract primitive operation - must be implemented by subclasses
    protected abstract method parseData(rawData: bytes): StructuredData

    // Hook operation - can be overridden but has default behavior
    protected method preprocessData(data: StructuredData): StructuredData {
        // Default: no preprocessing
        return data
    }

    // Concrete operation - shared implementation
    protected method analyzeData(data: StructuredData): AnalysisResult {
        result = new AnalysisResult()
        result.recordCount = data.rows.size()
        result.statistics = this.calculateStatistics(data)
        result.patterns = this.detectPatterns(data)
        return result
    }

    // Hook operation - empty default
    protected method postProcess(analysis: AnalysisResult): void {
        // Default: do nothing
    }

    // Concrete operation - shared implementation
    protected method generateReport(analysis: AnalysisResult): AnalysisReport {
        report = new AnalysisReport()
        report.summary = this.formatSummary(analysis)
        report.details = this.formatDetails(analysis)
        report.generatedAt = now()
        return report
    }

    // Concrete operation - shared implementation
    protected method closeFile(file: FileHandle): void {
        log("Closing file")
        file.close()
    }

    // Private helper methods
    private method calculateStatistics(data: StructuredData): Statistics { ... }
    private method detectPatterns(data: StructuredData): list<Pattern> { ... }
    private method formatSummary(analysis: AnalysisResult): string { ... }
    private method formatDetails(analysis: AnalysisResult): string { ... }
}
```

### Pseudocode: Concrete Implementations

```
// Concrete class for PDF documents
class PDFDataMiner extends DataMiner {
    private pdfReader: PDFReader

    constructor() {
        this.pdfReader = new PDFReader()
    }

    protected method extractData(file: FileHandle): bytes {
        log("Extracting data from PDF")
        document = this.pdfReader.loadDocument(file)
        text = ""

        for page in document.pages {
            text = text + page.extractText()
            // Also extract tables
            for table in page.extractTables() {
                text = text + table.toCSV()
            }
        }

        return text.toBytes()
    }

    protected method parseData(rawData: bytes): StructuredData {
        log("Parsing PDF text content")
        text = rawData.toString()
        lines = text.split("\n")

        data = new StructuredData()
        for line in lines {
            if line.isNotEmpty() {
                fields = this.parseLine(line)
                data.addRow(fields)
            }
        }

        return data
    }

    // Override hook to clean up PDF artifacts
    protected method preprocessData(data: StructuredData): StructuredData {
        log("Preprocessing PDF data - removing artifacts")
        cleaned = new StructuredData()

        for row in data.rows {
            // Remove header/footer artifacts
            if not this.isHeaderOrFooter(row) {
                cleaned.addRow(row)
            }
        }

        return cleaned
    }

    private method parseLine(line: string): list<string> { ... }
    private method isHeaderOrFooter(row: Row): boolean { ... }
}

// Concrete class for CSV files
class CSVDataMiner extends DataMiner {
    private delimiter: string
    private hasHeader: boolean

    constructor(delimiter: string = ",", hasHeader: boolean = true) {
        this.delimiter = delimiter
        this.hasHeader = hasHeader
    }

    protected method extractData(file: FileHandle): bytes {
        log("Extracting data from CSV")
        return file.readAll()
    }

    protected method parseData(rawData: bytes): StructuredData {
        log("Parsing CSV content")
        text = rawData.toString()
        lines = text.split("\n")

        data = new StructuredData()

        startIndex = 0
        if this.hasHeader {
            headers = this.parseCsvLine(lines[0])
            data.setHeaders(headers)
            startIndex = 1
        }

        for i = startIndex to lines.size() - 1 {
            if lines[i].trim().isNotEmpty() {
                fields = this.parseCsvLine(lines[i])
                data.addRow(fields)
            }
        }

        return data
    }

    private method parseCsvLine(line: string): list<string> {
        // Handle quoted fields, escaped delimiters, etc.
        fields = []
        currentField = ""
        inQuotes = false

        for char in line {
            if char == '"' {
                inQuotes = not inQuotes
            } else if char == this.delimiter and not inQuotes {
                fields.add(currentField.trim())
                currentField = ""
            } else {
                currentField = currentField + char
            }
        }
        fields.add(currentField.trim())

        return fields
    }

    // Use default preprocessData - CSV usually doesn't need it
    // Use default postProcess
}

// Concrete class for HTML documents
class HTMLDataMiner extends DataMiner {
    private parser: HTMLParser
    private tableSelector: string

    constructor(tableSelector: string = "table") {
        this.parser = new HTMLParser()
        this.tableSelector = tableSelector
    }

    protected method extractData(file: FileHandle): bytes {
        log("Extracting data from HTML")
        return file.readAll()
    }

    protected method parseData(rawData: bytes): StructuredData {
        log("Parsing HTML tables")
        html = rawData.toString()
        document = this.parser.parse(html)

        tables = document.querySelectorAll(this.tableSelector)
        data = new StructuredData()

        for table in tables {
            this.parseTable(table, data)
        }

        return data
    }

    private method parseTable(table: Element, data: StructuredData): void {
        rows = table.querySelectorAll("tr")

        // First row might be headers
        headerRow = rows[0]
        headers = headerRow.querySelectorAll("th")
        if headers.isNotEmpty() {
            data.setHeaders(headers.map(h => h.textContent))
        }

        // Parse data rows
        for i = (headers.isNotEmpty() ? 1 : 0) to rows.size() - 1 {
            cells = rows[i].querySelectorAll("td")
            fields = cells.map(c => this.cleanText(c.textContent))
            data.addRow(fields)
        }
    }

    // Override hook to strip HTML entities and clean text
    protected method preprocessData(data: StructuredData): StructuredData {
        log("Preprocessing HTML data - cleaning text")
        cleaned = new StructuredData()
        cleaned.setHeaders(data.headers)

        for row in data.rows {
            cleanedRow = row.map(field => this.decodeEntities(field))
            cleaned.addRow(cleanedRow)
        }

        return cleaned
    }

    // Override hook to log detailed statistics
    protected method postProcess(analysis: AnalysisResult): void {
        log("HTML Analysis complete:")
        log("  - Tables processed: " + analysis.metadata.get("tableCount"))
        log("  - Total rows: " + analysis.recordCount)
    }

    private method cleanText(text: string): string { ... }
    private method decodeEntities(text: string): string { ... }
}

// Concrete class for JSON data
class JSONDataMiner extends DataMiner {
    private jsonPath: string  // Path to extract data from JSON structure

    constructor(jsonPath: string = "$") {
        this.jsonPath = jsonPath
    }

    protected method extractData(file: FileHandle): bytes {
        log("Extracting data from JSON")
        return file.readAll()
    }

    protected method parseData(rawData: bytes): StructuredData {
        log("Parsing JSON content")
        json = JSON.parse(rawData.toString())

        // Navigate to the data using JSONPath
        targetData = this.evaluateJsonPath(json, this.jsonPath)

        data = new StructuredData()

        if targetData is list {
            // Infer headers from first object
            if targetData[0] is object {
                headers = targetData[0].keys().toList()
                data.setHeaders(headers)
            }

            // Convert each item to a row
            for item in targetData {
                if item is object {
                    row = headers.map(h => item.get(h, "").toString())
                    data.addRow(row)
                } else {
                    data.addRow([item.toString()])
                }
            }
        }

        return data
    }

    private method evaluateJsonPath(json: any, path: string): any { ... }
}
```

## Example

A complete example implementing a test framework with Template Method:

```
// Test result types
enum TestStatus {
    PASSED,
    FAILED,
    ERROR,
    SKIPPED
}

class TestResult {
    public name: string
    public status: TestStatus
    public duration: duration
    public message: string = null
    public exception: Exception = null
    public stdout: string = ""
    public stderr: string = ""
}

// Abstract test case - defines the test execution template
abstract class TestCase {
    protected name: string
    protected timeout: duration = Duration.ofSeconds(30)
    private result: TestResult

    constructor(name: string) {
        this.name = name
    }

    // Template method - defines the test execution algorithm
    final method run(): TestResult {
        this.result = new TestResult()
        this.result.name = this.name

        startTime = now()

        try {
            // Step 1: Check if test should be skipped
            if this.shouldSkip() {
                this.result.status = TestStatus.SKIPPED
                this.result.message = this.getSkipReason()
                return this.result
            }

            // Step 2: Set up test fixtures (hook - can be overridden)
            this.setUp()

            // Step 3: Set up class-level fixtures (hook)
            this.setUpClass()

            try {
                // Step 4: Run the actual test (abstract - must be implemented)
                this.runTest()

                // Step 5: Verify postconditions (hook)
                this.verifyPostconditions()

                // If we get here, test passed
                this.result.status = TestStatus.PASSED

            } finally {
                // Step 6: Tear down class-level fixtures (hook)
                this.tearDownClass()

                // Step 7: Tear down test fixtures (hook)
                this.tearDown()
            }

        } catch AssertionError as e {
            // Test failed
            this.result.status = TestStatus.FAILED
            this.result.message = e.message
            this.result.exception = e

        } catch TimeoutException as e {
            // Test timed out
            this.result.status = TestStatus.ERROR
            this.result.message = "Test timed out after " + this.timeout

        } catch Exception as e {
            // Unexpected error
            this.result.status = TestStatus.ERROR
            this.result.message = "Unexpected error: " + e.message
            this.result.exception = e
        }

        this.result.duration = now() - startTime
        return this.result
    }

    // Hook - can be overridden to skip test
    protected method shouldSkip(): boolean {
        return false
    }

    // Hook - reason for skipping
    protected method getSkipReason(): string {
        return "Test skipped"
    }

    // Hook - set up test fixtures
    protected method setUp(): void {
        // Default: do nothing
    }

    // Hook - set up class-level fixtures (once per class)
    protected method setUpClass(): void {
        // Default: do nothing
    }

    // Abstract - the actual test logic
    protected abstract method runTest(): void

    // Hook - verify postconditions after test
    protected method verifyPostconditions(): void {
        // Default: do nothing
    }

    // Hook - tear down class-level fixtures
    protected method tearDownClass(): void {
        // Default: do nothing
    }

    // Hook - tear down test fixtures
    protected method tearDown(): void {
        // Default: do nothing
    }

    // Assertion helpers
    protected method assertEqual(expected: any, actual: any, message: string = null): void {
        if expected != actual {
            msg = message ?? "Expected " + expected + " but got " + actual
            throw new AssertionError(msg)
        }
    }

    protected method assertTrue(condition: boolean, message: string = null): void {
        if not condition {
            throw new AssertionError(message ?? "Expected true but got false")
        }
    }

    protected method assertFalse(condition: boolean, message: string = null): void {
        if condition {
            throw new AssertionError(message ?? "Expected false but got true")
        }
    }

    protected method assertNull(value: any, message: string = null): void {
        if value is not null {
            throw new AssertionError(message ?? "Expected null but got " + value)
        }
    }

    protected method assertNotNull(value: any, message: string = null): void {
        if value is null {
            throw new AssertionError(message ?? "Expected non-null value")
        }
    }

    protected method assertThrows(exceptionType: Type, action: function): void {
        try {
            action()
            throw new AssertionError("Expected " + exceptionType.name + " but no exception thrown")
        } catch Exception as e {
            if not (e is exceptionType) {
                throw new AssertionError(
                    "Expected " + exceptionType.name + " but got " + e.getClass().name
                )
            }
        }
    }

    protected method fail(message: string): void {
        throw new AssertionError(message)
    }
}

// Concrete test case for testing a calculator
class CalculatorTest extends TestCase {
    private calculator: Calculator

    constructor(name: string) {
        super(name)
    }

    // Set up runs before each test
    protected method setUp(): void {
        this.calculator = new Calculator()
        log("Calculator initialized")
    }

    // Tear down runs after each test
    protected method tearDown(): void {
        this.calculator = null
        log("Calculator cleaned up")
    }
}

// Specific test implementations
class AdditionTest extends CalculatorTest {
    constructor() {
        super("Addition Test")
    }

    protected method runTest(): void {
        result = this.calculator.add(2, 3)
        this.assertEqual(5, result, "2 + 3 should equal 5")

        result = this.calculator.add(-1, 1)
        this.assertEqual(0, result, "-1 + 1 should equal 0")

        result = this.calculator.add(0, 0)
        this.assertEqual(0, result, "0 + 0 should equal 0")
    }
}

class DivisionTest extends CalculatorTest {
    constructor() {
        super("Division Test")
    }

    protected method runTest(): void {
        result = this.calculator.divide(10, 2)
        this.assertEqual(5, result, "10 / 2 should equal 5")

        result = this.calculator.divide(7, 2)
        this.assertEqual(3.5, result, "7 / 2 should equal 3.5")
    }

    // Verify no side effects
    protected method verifyPostconditions(): void {
        this.assertNotNull(this.calculator, "Calculator should still exist")
    }
}

class DivisionByZeroTest extends CalculatorTest {
    constructor() {
        super("Division By Zero Test")
    }

    protected method runTest(): void {
        this.assertThrows(ArithmeticException, () => {
            this.calculator.divide(10, 0)
        })
    }
}

// Test that requires database - shows skip functionality
class DatabaseIntegrationTest extends TestCase {
    private database: Database

    constructor() {
        super("Database Integration Test")
    }

    protected method shouldSkip(): boolean {
        // Skip if database is not available
        return not Database.isAvailable()
    }

    protected method getSkipReason(): string {
        return "Database not available"
    }

    protected method setUp(): void {
        this.database = Database.connect()
        this.database.beginTransaction()
    }

    protected method runTest(): void {
        // Test database operations
        this.database.execute("INSERT INTO users (name) VALUES ('test')")
        result = this.database.query("SELECT * FROM users WHERE name = 'test'")
        this.assertEqual(1, result.rows.size())
    }

    protected method tearDown(): void {
        // Rollback to not affect other tests
        this.database.rollback()
        this.database.disconnect()
    }
}

// Test runner that uses the test cases
class TestRunner {
    private tests: list<TestCase> = []
    private results: list<TestResult> = []

    method addTest(test: TestCase): void {
        this.tests.add(test)
    }

    method runAll(): TestSuiteResult {
        this.results = []
        startTime = now()

        for test in this.tests {
            log("\nRunning: " + test.name)
            result = test.run()
            this.results.add(result)
            this.printResult(result)
        }

        return this.summarize(now() - startTime)
    }

    private method printResult(result: TestResult): void {
        switch result.status {
            case TestStatus.PASSED:
                print("  PASSED (" + result.duration.toMillis() + "ms)")
            case TestStatus.FAILED:
                print("  FAILED: " + result.message)
            case TestStatus.ERROR:
                print("  ERROR: " + result.message)
            case TestStatus.SKIPPED:
                print("  SKIPPED: " + result.message)
        }
    }

    private method summarize(totalDuration: duration): TestSuiteResult {
        passed = this.results.filter(r => r.status == TestStatus.PASSED).size()
        failed = this.results.filter(r => r.status == TestStatus.FAILED).size()
        errors = this.results.filter(r => r.status == TestStatus.ERROR).size()
        skipped = this.results.filter(r => r.status == TestStatus.SKIPPED).size()

        print("\n========================================")
        print("Results: " + passed + " passed, " + failed + " failed, " +
              errors + " errors, " + skipped + " skipped")
        print("Duration: " + totalDuration.toMillis() + "ms")
        print("========================================")

        return new TestSuiteResult(
            total: this.results.size(),
            passed: passed,
            failed: failed,
            errors: errors,
            skipped: skipped,
            duration: totalDuration,
            results: this.results
        )
    }
}

// Usage example
function main() {
    runner = new TestRunner()

    // Add tests
    runner.addTest(new AdditionTest())
    runner.addTest(new DivisionTest())
    runner.addTest(new DivisionByZeroTest())
    runner.addTest(new DatabaseIntegrationTest())

    // Run all tests
    suiteResult = runner.runAll()

    // Exit with appropriate code
    if suiteResult.failed > 0 or suiteResult.errors > 0 {
        exit(1)
    }
    exit(0)
}

/*
Output:
Running: Addition Test
  PASSED (5ms)

Running: Division Test
  PASSED (3ms)

Running: Division By Zero Test
  PASSED (2ms)

Running: Database Integration Test
  SKIPPED: Database not available

========================================
Results: 3 passed, 0 failed, 0 errors, 1 skipped
Duration: 15ms
========================================
*/
```

## Known Uses

- **JUnit/TestNG**: The `setUp()`, `tearDown()`, `setUpClass()`, `tearDownClass()` methods in testing frameworks are classic Template Method implementations.

- **Java I/O Streams**: `InputStream.read(byte[], int, int)` calls the abstract `read()` method.

- **Servlet Lifecycle**: `HttpServlet.service()` calls `doGet()`, `doPost()`, etc.

- **React Component Lifecycle**: `componentDidMount()`, `componentWillUnmount()` are hooks in the component lifecycle template.

- **Django Class-Based Views**: `get()`, `post()`, `dispatch()` follow the Template Method pattern.

- **Spring Data Repositories**: Custom repository implementations use template methods for database operations.

- **Android Activity Lifecycle**: `onCreate()`, `onStart()`, `onResume()` are hooks in the activity template.

- **Compiler Passes**: Many compilers use template methods for parse-analyze-optimize-generate phases.

- **Build Systems**: Maven's build lifecycle (validate, compile, test, package, verify, install, deploy) is a template.

## Related Patterns

- **Strategy**: Both allow algorithm variation. Template Method uses inheritance and varies parts of an algorithm; Strategy uses composition and varies the entire algorithm.

- **Factory Method**: Factory Method is often called by Template Methods to create objects needed for algorithm steps.

- **Hook Methods**: Hooks are a specific implementation technique within Template Method that provides optional extension points.

## When NOT to Use

1. **When steps vary significantly**: If subclasses need radically different algorithms, not just different step implementations, composition (Strategy) is better.

```
// Poor fit: Algorithm structure differs significantly
abstract class Processor {
    method process() {  // Template method
        step1()
        step2()
        step3()
    }
}

class TypeAProcessor extends Processor {
    // Needs: step1, step2, step3 - fits template
}

class TypeBProcessor extends Processor {
    // Needs: stepX, step2, stepY, stepZ - doesn't fit!
    // Forced to use empty step1, step3, add hooks for X, Y, Z
}
```

2. **When inheritance hierarchies become deep**: Deep hierarchies with nested template methods become hard to understand and maintain.

3. **When runtime algorithm switching is needed**: Template Method fixes the algorithm at compile time via inheritance. Use Strategy for runtime switching.

4. **Single implementation**: If there's only one way to implement the algorithm, the Template Method infrastructure is overkill.

5. **When hooks proliferate**: If you have more hooks than concrete steps, the template becomes unclear. Consider other patterns.

```
// Too many hooks - template method loses clarity
abstract class Processor {
    method process() {
        beforeStart()       // hook
        start()             // hook
        afterStart()        // hook
        beforeProcess()     // hook
        process()           // abstract
        afterProcess()      // hook
        beforeFinish()      // hook
        finish()            // hook
        afterFinish()       // hook
    }
}
```

6. **When composition would be simpler**: Modern design often favors composition over inheritance. Consider injecting strategy objects instead of subclassing.

The Template Method pattern excels when you have a fixed algorithm structure with variable steps. Use it for frameworks, lifecycle management, and standardized processes. Avoid it when flexibility, runtime configuration, or flat hierarchies are more important.

---

*Based on concepts from "Design Patterns: Elements of Reusable Object-Oriented Software" by Gamma, Helm, Johnson, and Vlissides (Gang of Four), 1994.*
