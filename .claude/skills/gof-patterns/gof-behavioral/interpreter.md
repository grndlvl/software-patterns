# Interpreter

## Intent

Given a language, define a representation for its grammar along with an interpreter that uses the representation to interpret sentences in the language.

## Also Known As

- Expression Evaluator
- Language Processor

## Motivation

Consider a search engine that needs to support complex query expressions. Users want to search with queries like "title:javascript AND (author:smith OR year:2024) NOT draft:true". Implementing this with hardcoded conditionals would be unwieldy and impossible to extend. Each query needs to be parsed into a structure that can be evaluated against documents.

The Interpreter pattern addresses this by defining a grammar for the query language and representing each grammar rule as a class. The expression "title:javascript AND author:smith" becomes a tree of objects: an AndExpression containing a FieldMatchExpression for "title:javascript" and another for "author:smith". Each expression class knows how to interpret (evaluate) itself given a context (the document being searched).

This approach makes it easy to add new expression types without changing existing code. Want to add a NEAR operator for proximity searches? Just create a NearExpression class. The grammar is explicit in the class hierarchy, making it self-documenting. And because expressions are objects, they can be manipulated programmatically—optimized, cached, or serialized for later use.

The pattern works beautifully for domain-specific languages (DSLs) where the grammar is relatively simple and the expressions need to be evaluated repeatedly against different contexts. It's less suitable for complex, general-purpose languages where traditional parsing techniques and compilers are more appropriate.

## Applicability

Use the Interpreter pattern when:

- The grammar is simple. The pattern works best for grammars that can be represented with a manageable class hierarchy. Complex grammars become unwieldy.
- Efficiency is not a critical concern. Interpreter pattern is rarely the most efficient approach, but it's often efficient enough for its use cases.
- You need to interpret expressions repeatedly. The overhead of building the expression tree pays off when the same expression is evaluated many times.
- You want the grammar to be extensible. New expression types can be added as new classes without modifying existing code.
- The language maps naturally to a composite structure. Many DSLs consist of primitive expressions combined with composite expressions.

Common applications include:
- Search query languages
- Configuration file parsers
- Rule engines
- Mathematical expression evaluators
- Regular expression matchers
- SQL WHERE clause evaluation
- Template languages
- Validation rule languages

## Structure

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│    ┌────────┐         ┌───────────────────────────────────────────┐        │
│    │ Client │────────>│           AbstractExpression              │        │
│    └────────┘         │              <<interface>>                │        │
│         │             ├───────────────────────────────────────────┤        │
│         │             │ + interpret(context: Context): Result     │        │
│         │             └───────────────────────────────────────────┘        │
│         │                               △                                   │
│         │                               │                                   │
│         │             ┌─────────────────┴─────────────────────┐            │
│         │             │                                       │            │
│         │    ┌────────┴────────────┐          ┌──────────────┴──────────┐ │
│         │    │ TerminalExpression  │          │  NonterminalExpression  │ │
│         │    ├─────────────────────┤          ├─────────────────────────┤ │
│         │    │                     │          │ - expressions: list     │ │
│         │    ├─────────────────────┤          ├─────────────────────────┤ │
│         │    │ + interpret(ctx)    │          │ + interpret(ctx)        │ │
│         │    │   // evaluates      │          │   // combines child     │ │
│         │    │   // terminal       │          │   // interpretations    │ │
│         │    └─────────────────────┘          └─────────────────────────┘ │
│         │                                                │                 │
│         │                                                │ uses            │
│         ▼                                                ▼                 │
│    ┌─────────┐                              ┌───────────────────────┐     │
│    │ Context │                              │  AbstractExpression   │     │
│    ├─────────┤                              │  (children)           │     │
│    │ - state │                              └───────────────────────┘     │
│    │ - vars  │                                                             │
│    └─────────┘                                                             │
│                                                                             │
│   Abstract Syntax Tree Example:                                            │
│                                                                             │
│            ┌─────┐                                                         │
│            │ AND │  (NonterminalExpression)                                │
│            └──┬──┘                                                         │
│         ┌────┴────┐                                                        │
│         │         │                                                        │
│      ┌──┴──┐   ┌──┴──┐                                                    │
│      │ OR  │   │ NOT │  (NonterminalExpressions)                          │
│      └──┬──┘   └──┬──┘                                                    │
│      ┌──┴──┐      │                                                        │
│      │     │      │                                                        │
│    ┌─┴─┐ ┌─┴─┐  ┌─┴─┐                                                     │
│    │ a │ │ b │  │ c │   (TerminalExpressions)                             │
│    └───┘ └───┘  └───┘                                                     │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Participants

- **AbstractExpression**: Declares an abstract `interpret` operation that is common to all nodes in the abstract syntax tree.

- **TerminalExpression**: Implements an `interpret` operation for terminal symbols in the grammar. These are the leaf nodes that don't contain other expressions. One instance required for each terminal symbol in a sentence.

- **NonterminalExpression**: Implements `interpret` for nonterminal symbols. Contains references to other AbstractExpression objects (its children). Combines the interpretations of its children according to the grammar rule it represents.

- **Context**: Contains information that's global to the interpreter—variable values, input data, or any state needed during interpretation.

- **Client**: Builds (or is given) the abstract syntax tree representing a sentence in the language. Invokes the `interpret` operation.

## Collaborations

1. The client builds (or receives) a sentence represented as an abstract syntax tree of TerminalExpression and NonterminalExpression instances.

2. The client initializes a context with any necessary information (variable bindings, input data, etc.).

3. The client invokes the `interpret` operation on the root of the tree, passing the context.

4. Each NonterminalExpression node interprets by recursively calling `interpret` on its children and combining the results according to its grammatical rule.

5. Each TerminalExpression node interprets by directly evaluating against the context.

6. The interpret operations use the context to store and access state during interpretation (e.g., variable values, intermediate results).

## Consequences

### Benefits

1. **Easy to change and extend the grammar**: Because the grammar is represented by classes, you can use inheritance to extend it. Existing expression classes don't need to change when you add new ones.

2. **Implementing the grammar is straightforward**: Classes defining nodes in the abstract syntax tree have similar implementations. They're easy to write and often can be automated with parser generators.

3. **Adding new ways to interpret expressions**: The Interpreter pattern makes it easy to evaluate expressions in new ways. You can define new operations on expressions by adding new interpret methods (using Visitor pattern) or by creating alternative expression hierarchies.

4. **Self-documenting grammar**: The class hierarchy explicitly represents the grammar, making the language's structure clear to maintainers.

5. **Expressions are reusable**: Once built, expression trees can be evaluated repeatedly against different contexts without reparsing.

### Liabilities

1. **Complex grammars are hard to maintain**: The Interpreter pattern defines at least one class for every rule in the grammar. Grammars containing many rules can be hard to manage and maintain. Other techniques (parser generators) are more appropriate for complex grammars.

2. **Not efficient for complex languages**: Deeply nested or heavily recursive grammars create deep expression trees with significant interpretation overhead. Compilation to bytecode or machine code is more efficient for complex languages.

3. **Parsing is not addressed**: The pattern assumes you already have an abstract syntax tree. Building that tree from text requires a separate parser, which can be more work than the interpreter itself.

## Implementation

### Implementation Considerations

1. **Creating the abstract syntax tree**: The Interpreter pattern doesn't specify how to create the AST. Common approaches:
   - Hand-written recursive descent parser
   - Parser generator (ANTLR, PEG.js, etc.)
   - Parser combinator library
   - Simple tokenizer with operator precedence parsing

2. **Defining the interpret operation**: The interpret operation can return different types depending on the language:
   - Boolean for predicate languages (filters, validators)
   - Number for arithmetic expressions
   - String for template languages
   - Any type for general expression languages

3. **Sharing terminal symbols**: Terminal expressions for the same literal value (e.g., the number 5) can be shared using Flyweight pattern to save memory.

4. **Using Visitor for multiple operations**: If you need multiple operations on the expression tree (interpret, print, optimize), consider using the Visitor pattern instead of adding methods to each expression class.

### Pseudocode: Grammar Definition

```
// Grammar for a simple arithmetic expression language:
//
// expression  ::= term (('+' | '-') term)*
// term        ::= factor (('*' | '/') factor)*
// factor      ::= number | '(' expression ')' | variable
// number      ::= [0-9]+
// variable    ::= [a-z]+

// Context holds variable bindings
class Context {
    private variables: map<string, number> = {}

    method setVariable(name: string, value: number): void {
        this.variables.set(name, value)
    }

    method getVariable(name: string): number {
        if not this.variables.containsKey(name) {
            throw "Undefined variable: " + name
        }
        return this.variables.get(name)
    }

    method hasVariable(name: string): boolean {
        return this.variables.containsKey(name)
    }
}

// Abstract expression interface
interface Expression {
    method interpret(context: Context): number
    method toString(): string
}
```

### Pseudocode: Terminal Expressions

```
// Terminal: Number literal
class NumberExpression implements Expression {
    private value: number

    constructor(value: number) {
        this.value = value
    }

    method interpret(context: Context): number {
        return this.value
    }

    method toString(): string {
        return this.value.toString()
    }
}

// Terminal: Variable reference
class VariableExpression implements Expression {
    private name: string

    constructor(name: string) {
        this.name = name
    }

    method interpret(context: Context): number {
        return context.getVariable(this.name)
    }

    method toString(): string {
        return this.name
    }
}
```

### Pseudocode: Nonterminal Expressions

```
// Nonterminal: Addition
class AddExpression implements Expression {
    private left: Expression
    private right: Expression

    constructor(left: Expression, right: Expression) {
        this.left = left
        this.right = right
    }

    method interpret(context: Context): number {
        return this.left.interpret(context) + this.right.interpret(context)
    }

    method toString(): string {
        return "(" + this.left.toString() + " + " + this.right.toString() + ")"
    }
}

// Nonterminal: Subtraction
class SubtractExpression implements Expression {
    private left: Expression
    private right: Expression

    constructor(left: Expression, right: Expression) {
        this.left = left
        this.right = right
    }

    method interpret(context: Context): number {
        return this.left.interpret(context) - this.right.interpret(context)
    }

    method toString(): string {
        return "(" + this.left.toString() + " - " + this.right.toString() + ")"
    }
}

// Nonterminal: Multiplication
class MultiplyExpression implements Expression {
    private left: Expression
    private right: Expression

    constructor(left: Expression, right: Expression) {
        this.left = left
        this.right = right
    }

    method interpret(context: Context): number {
        return this.left.interpret(context) * this.right.interpret(context)
    }

    method toString(): string {
        return "(" + this.left.toString() + " * " + this.right.toString() + ")"
    }
}

// Nonterminal: Division
class DivideExpression implements Expression {
    private left: Expression
    private right: Expression

    constructor(left: Expression, right: Expression) {
        this.left = left
        this.right = right
    }

    method interpret(context: Context): number {
        divisor = this.right.interpret(context)
        if divisor == 0 {
            throw "Division by zero"
        }
        return this.left.interpret(context) / divisor
    }

    method toString(): string {
        return "(" + this.left.toString() + " / " + this.right.toString() + ")"
    }
}

// Nonterminal: Negation (unary)
class NegateExpression implements Expression {
    private operand: Expression

    constructor(operand: Expression) {
        this.operand = operand
    }

    method interpret(context: Context): number {
        return -this.operand.interpret(context)
    }

    method toString(): string {
        return "-" + this.operand.toString()
    }
}
```

### Pseudocode: Parser (builds the AST)

```
class ExpressionParser {
    private tokens: list<Token>
    private position: int = 0

    method parse(input: string): Expression {
        this.tokens = tokenize(input)
        this.position = 0
        expression = this.parseExpression()

        if this.position < this.tokens.size() {
            throw "Unexpected token: " + this.currentToken().value
        }

        return expression
    }

    private method parseExpression(): Expression {
        left = this.parseTerm()

        while this.hasMore() and this.currentToken().type in [PLUS, MINUS] {
            operator = this.consume()
            right = this.parseTerm()

            if operator.type == PLUS {
                left = new AddExpression(left, right)
            } else {
                left = new SubtractExpression(left, right)
            }
        }

        return left
    }

    private method parseTerm(): Expression {
        left = this.parseFactor()

        while this.hasMore() and this.currentToken().type in [MULTIPLY, DIVIDE] {
            operator = this.consume()
            right = this.parseFactor()

            if operator.type == MULTIPLY {
                left = new MultiplyExpression(left, right)
            } else {
                left = new DivideExpression(left, right)
            }
        }

        return left
    }

    private method parseFactor(): Expression {
        token = this.currentToken()

        if token.type == NUMBER {
            this.consume()
            return new NumberExpression(parseFloat(token.value))
        }

        if token.type == VARIABLE {
            this.consume()
            return new VariableExpression(token.value)
        }

        if token.type == MINUS {
            this.consume()
            return new NegateExpression(this.parseFactor())
        }

        if token.type == LEFT_PAREN {
            this.consume()  // consume '('
            expression = this.parseExpression()
            this.expect(RIGHT_PAREN)
            return expression
        }

        throw "Unexpected token: " + token.value
    }

    private method currentToken(): Token {
        return this.tokens.get(this.position)
    }

    private method consume(): Token {
        token = this.currentToken()
        this.position = this.position + 1
        return token
    }

    private method expect(type: TokenType): void {
        if this.currentToken().type != type {
            throw "Expected " + type + " but got " + this.currentToken().type
        }
        this.consume()
    }

    private method hasMore(): boolean {
        return this.position < this.tokens.size()
    }
}
```

## Example

A complete example implementing a query language for filtering data:

```
// Domain: Product catalog with search/filter capabilities

class Product {
    public id: string
    public name: string
    public category: string
    public price: number
    public rating: number
    public inStock: boolean
    public tags: list<string>
    public attributes: map<string, any>

    method getAttribute(path: string): any {
        // Support nested paths like "dimensions.width"
        parts = path.split(".")
        current = this.attributes

        for part in parts {
            if current is map {
                current = current.get(part)
            } else {
                return null
            }
        }
        return current
    }
}

// Context for query interpretation
class QueryContext {
    private product: Product

    constructor(product: Product) {
        this.product = product
    }

    method getProduct(): Product {
        return this.product
    }

    method getFieldValue(field: string): any {
        switch field {
            case "id": return this.product.id
            case "name": return this.product.name
            case "category": return this.product.category
            case "price": return this.product.price
            case "rating": return this.product.rating
            case "inStock": return this.product.inStock
            case "tags": return this.product.tags
            default: return this.product.getAttribute(field)
        }
    }
}

// Abstract expression for query predicates
interface QueryExpression {
    method interpret(context: QueryContext): boolean
    method toString(): string
}

// Terminal: Field equals value
class EqualsExpression implements QueryExpression {
    private field: string
    private value: any

    constructor(field: string, value: any) {
        this.field = field
        this.value = value
    }

    method interpret(context: QueryContext): boolean {
        fieldValue = context.getFieldValue(this.field)
        return fieldValue == this.value
    }

    method toString(): string {
        return this.field + " = " + quote(this.value)
    }
}

// Terminal: Field contains value (for strings and arrays)
class ContainsExpression implements QueryExpression {
    private field: string
    private value: any

    constructor(field: string, value: any) {
        this.field = field
        this.value = value
    }

    method interpret(context: QueryContext): boolean {
        fieldValue = context.getFieldValue(this.field)

        if fieldValue is string {
            return fieldValue.toLowerCase().contains(this.value.toLowerCase())
        }

        if fieldValue is list {
            return this.value in fieldValue
        }

        return false
    }

    method toString(): string {
        return this.field + " CONTAINS " + quote(this.value)
    }
}

// Terminal: Numeric comparison
class ComparisonExpression implements QueryExpression {
    private field: string
    private operator: string  // "<", "<=", ">", ">="
    private value: number

    constructor(field: string, operator: string, value: number) {
        this.field = field
        this.operator = operator
        this.value = value
    }

    method interpret(context: QueryContext): boolean {
        fieldValue = context.getFieldValue(this.field)

        if fieldValue is not number {
            return false
        }

        switch this.operator {
            case "<":  return fieldValue < this.value
            case "<=": return fieldValue <= this.value
            case ">":  return fieldValue > this.value
            case ">=": return fieldValue >= this.value
            default:   return false
        }
    }

    method toString(): string {
        return this.field + " " + this.operator + " " + this.value
    }
}

// Terminal: Range check (BETWEEN)
class RangeExpression implements QueryExpression {
    private field: string
    private min: number
    private max: number

    constructor(field: string, min: number, max: number) {
        this.field = field
        this.min = min
        this.max = max
    }

    method interpret(context: QueryContext): boolean {
        fieldValue = context.getFieldValue(this.field)

        if fieldValue is not number {
            return false
        }

        return fieldValue >= this.min and fieldValue <= this.max
    }

    method toString(): string {
        return this.field + " BETWEEN " + this.min + " AND " + this.max
    }
}

// Terminal: Pattern matching (LIKE)
class LikeExpression implements QueryExpression {
    private field: string
    private pattern: string
    private regex: Regex

    constructor(field: string, pattern: string) {
        this.field = field
        this.pattern = pattern
        // Convert SQL-like pattern to regex
        // % matches any sequence, _ matches single character
        regexPattern = pattern
            .replace(".", "\\.")
            .replace("%", ".*")
            .replace("_", ".")
        this.regex = new Regex("^" + regexPattern + "$", "i")
    }

    method interpret(context: QueryContext): boolean {
        fieldValue = context.getFieldValue(this.field)

        if fieldValue is not string {
            return false
        }

        return this.regex.test(fieldValue)
    }

    method toString(): string {
        return this.field + " LIKE '" + this.pattern + "'"
    }
}

// Terminal: IN list check
class InExpression implements QueryExpression {
    private field: string
    private values: list<any>

    constructor(field: string, values: list<any>) {
        this.field = field
        this.values = values
    }

    method interpret(context: QueryContext): boolean {
        fieldValue = context.getFieldValue(this.field)
        return fieldValue in this.values
    }

    method toString(): string {
        return this.field + " IN (" + join(this.values, ", ") + ")"
    }
}

// Terminal: NULL check
class IsNullExpression implements QueryExpression {
    private field: string
    private checkNull: boolean  // true for IS NULL, false for IS NOT NULL

    constructor(field: string, checkNull: boolean) {
        this.field = field
        this.checkNull = checkNull
    }

    method interpret(context: QueryContext): boolean {
        fieldValue = context.getFieldValue(this.field)
        isNull = fieldValue is null or fieldValue is undefined

        return this.checkNull ? isNull : not isNull
    }

    method toString(): string {
        return this.field + (this.checkNull ? " IS NULL" : " IS NOT NULL")
    }
}

// Nonterminal: AND
class AndExpression implements QueryExpression {
    private left: QueryExpression
    private right: QueryExpression

    constructor(left: QueryExpression, right: QueryExpression) {
        this.left = left
        this.right = right
    }

    method interpret(context: QueryContext): boolean {
        // Short-circuit evaluation
        if not this.left.interpret(context) {
            return false
        }
        return this.right.interpret(context)
    }

    method toString(): string {
        return "(" + this.left.toString() + " AND " + this.right.toString() + ")"
    }
}

// Nonterminal: OR
class OrExpression implements QueryExpression {
    private left: QueryExpression
    private right: QueryExpression

    constructor(left: QueryExpression, right: QueryExpression) {
        this.left = left
        this.right = right
    }

    method interpret(context: QueryContext): boolean {
        // Short-circuit evaluation
        if this.left.interpret(context) {
            return true
        }
        return this.right.interpret(context)
    }

    method toString(): string {
        return "(" + this.left.toString() + " OR " + this.right.toString() + ")"
    }
}

// Nonterminal: NOT
class NotExpression implements QueryExpression {
    private operand: QueryExpression

    constructor(operand: QueryExpression) {
        this.operand = operand
    }

    method interpret(context: QueryContext): boolean {
        return not this.operand.interpret(context)
    }

    method toString(): string {
        return "NOT " + this.operand.toString()
    }
}

// Query builder with fluent API
class QueryBuilder {
    static method field(name: string): FieldBuilder {
        return new FieldBuilder(name)
    }

    static method and(left: QueryExpression, right: QueryExpression): QueryExpression {
        return new AndExpression(left, right)
    }

    static method or(left: QueryExpression, right: QueryExpression): QueryExpression {
        return new OrExpression(left, right)
    }

    static method not(expr: QueryExpression): QueryExpression {
        return new NotExpression(expr)
    }

    static method all(expressions: list<QueryExpression>): QueryExpression {
        if expressions.isEmpty() {
            return new TrueExpression()
        }
        result = expressions.get(0)
        for i = 1 to expressions.size() - 1 {
            result = new AndExpression(result, expressions.get(i))
        }
        return result
    }

    static method any(expressions: list<QueryExpression>): QueryExpression {
        if expressions.isEmpty() {
            return new FalseExpression()
        }
        result = expressions.get(0)
        for i = 1 to expressions.size() - 1 {
            result = new OrExpression(result, expressions.get(i))
        }
        return result
    }
}

class FieldBuilder {
    private field: string

    constructor(field: string) {
        this.field = field
    }

    method equals(value: any): QueryExpression {
        return new EqualsExpression(this.field, value)
    }

    method contains(value: any): QueryExpression {
        return new ContainsExpression(this.field, value)
    }

    method lessThan(value: number): QueryExpression {
        return new ComparisonExpression(this.field, "<", value)
    }

    method lessOrEqual(value: number): QueryExpression {
        return new ComparisonExpression(this.field, "<=", value)
    }

    method greaterThan(value: number): QueryExpression {
        return new ComparisonExpression(this.field, ">", value)
    }

    method greaterOrEqual(value: number): QueryExpression {
        return new ComparisonExpression(this.field, ">=", value)
    }

    method between(min: number, max: number): QueryExpression {
        return new RangeExpression(this.field, min, max)
    }

    method like(pattern: string): QueryExpression {
        return new LikeExpression(this.field, pattern)
    }

    method in(values: list<any>): QueryExpression {
        return new InExpression(this.field, values)
    }

    method isNull(): QueryExpression {
        return new IsNullExpression(this.field, true)
    }

    method isNotNull(): QueryExpression {
        return new IsNullExpression(this.field, false)
    }
}

// Product catalog that uses the query language
class ProductCatalog {
    private products: list<Product> = []

    method add(product: Product): void {
        this.products.add(product)
    }

    method search(query: QueryExpression): list<Product> {
        results = []

        for product in this.products {
            context = new QueryContext(product)
            if query.interpret(context) {
                results.add(product)
            }
        }

        return results
    }

    method count(query: QueryExpression): int {
        count = 0

        for product in this.products {
            context = new QueryContext(product)
            if query.interpret(context) {
                count = count + 1
            }
        }

        return count
    }
}

// Usage example
function main() {
    catalog = new ProductCatalog()

    // Add some products
    catalog.add(new Product(
        id: "1",
        name: "JavaScript: The Good Parts",
        category: "Books",
        price: 29.99,
        rating: 4.5,
        inStock: true,
        tags: ["programming", "javascript", "web"]
    ))

    catalog.add(new Product(
        id: "2",
        name: "Mechanical Keyboard",
        category: "Electronics",
        price: 149.99,
        rating: 4.8,
        inStock: true,
        tags: ["keyboard", "gaming", "rgb"]
    ))

    catalog.add(new Product(
        id: "3",
        name: "Python Crash Course",
        category: "Books",
        price: 35.99,
        rating: 4.2,
        inStock: false,
        tags: ["programming", "python", "beginner"]
    ))

    // Build queries using the fluent builder
    Q = QueryBuilder

    // Query 1: Books under $35
    query1 = Q.and(
        Q.field("category").equals("Books"),
        Q.field("price").lessThan(35)
    )
    results1 = catalog.search(query1)
    // Returns: JavaScript: The Good Parts

    // Query 2: In-stock items with rating >= 4.5
    query2 = Q.and(
        Q.field("inStock").equals(true),
        Q.field("rating").greaterOrEqual(4.5)
    )
    results2 = catalog.search(query2)
    // Returns: JavaScript: The Good Parts, Mechanical Keyboard

    // Query 3: Programming books OR electronics
    query3 = Q.or(
        Q.and(
            Q.field("category").equals("Books"),
            Q.field("tags").contains("programming")
        ),
        Q.field("category").equals("Electronics")
    )
    results3 = catalog.search(query3)
    // Returns: All three products

    // Query 4: Name matches pattern
    query4 = Q.field("name").like("%Keyboard%")
    results4 = catalog.search(query4)
    // Returns: Mechanical Keyboard

    // Query 5: Complex query with NOT
    query5 = Q.and(
        Q.field("price").between(25, 50),
        Q.not(Q.field("inStock").equals(false))
    )
    results5 = catalog.search(query5)
    // Returns: JavaScript: The Good Parts

    // Print the query (self-documenting)
    print(query5.toString())
    // Output: (price BETWEEN 25 AND 50 AND NOT (inStock = false))
}
```

## Known Uses

- **Regular Expressions**: Most regex engines parse patterns into expression trees that are interpreted against input strings.

- **SQL WHERE Clauses**: Database query optimizers parse WHERE clauses into expression trees for evaluation and optimization.

- **Spring Expression Language (SpEL)**: Spring Framework's expression language for configuration and runtime evaluation.

- **JEXL (Java Expression Language)**: Apache Commons library for evaluating expressions in Java applications.

- **ANTLR-based Interpreters**: Many DSLs built with ANTLR use the Interpreter pattern for their generated parsers.

- **Business Rule Engines**: Drools, Easy Rules, and similar engines represent rules as interpretable expressions.

- **Template Engines**: Handlebars, Mustache, and Jinja parse templates into ASTs that are interpreted to produce output.

- **Mathematical Software**: Mathematica, MATLAB, and scientific calculators interpret mathematical expressions.

- **Configuration Languages**: Terraform HCL, Kubernetes YAML with expressions, and similar tools interpret configuration expressions.

## Related Patterns

- **Composite**: The abstract syntax tree is an instance of the Composite pattern. Terminal expressions are leaves; nonterminal expressions are composites.

- **Flyweight**: Terminal expressions representing the same literal value can be shared to save memory, using the Flyweight pattern.

- **Iterator**: The interpreter can use an Iterator to traverse the AST, although recursive traversal is more common.

- **Visitor**: If you need multiple operations on the expression tree (interpret, print, optimize, serialize), Visitor lets you add operations without changing expression classes.

- **Factory Method**: A parser can use Factory Method to create appropriate expression objects based on tokens.

- **Builder**: Complex expression trees can be constructed using a Builder, as shown in the QueryBuilder example.

## When NOT to Use

1. **Complex grammars**: If your language has dozens of grammar rules, the class explosion becomes unmanageable. Use a parser generator (ANTLR, Bison, PEG.js) and a more traditional interpreter or compiler architecture.

```
// Anti-pattern: Interpreter for a complex language
// Don't create 100+ expression classes for a real programming language
class IfStatementExpression {}
class WhileLoopExpression {}
class ForLoopExpression {}
class FunctionDeclarationExpression {}
class ClassDeclarationExpression {}
// ... 95 more classes

// Use a proper compiler/interpreter architecture instead
```

2. **Performance-critical applications**: Interpreting an AST is inherently slower than compiled code. For hot paths, consider compiling to bytecode or native code.

3. **When parsing dominates**: If most of the work is in parsing and the interpretation is trivial, the pattern may be overkill. A simple parser that directly produces results may suffice.

4. **One-shot evaluation**: If each expression is parsed and evaluated only once, the overhead of building an AST isn't justified.

```
// Overkill: Building AST for one-time evaluation
expression = parser.parse("2 + 3")  // Creates AST
result = expression.interpret(context)  // Traverses AST
// AST is never used again

// Just evaluate directly during parsing
result = parser.parseAndEvaluate("2 + 3")
```

5. **When the language changes frequently**: Every grammar change requires modifying the class hierarchy. If your DSL is evolving rapidly, a more dynamic approach (data-driven rules, configuration) may be more maintainable.

6. **Simple conditional logic**: For basic if/else logic, don't create an expression language—just write the conditionals directly.

```
// Over-engineering: Query language for simple filtering
query = parser.parse("status = 'active' AND type = 'premium'")
users = catalog.search(query)

// Just use a simple predicate function
users = catalog.filter(user =>
    user.status == "active" and user.type == "premium"
)
```

The Interpreter pattern excels at well-defined, stable DSLs with relatively simple grammars that need to be evaluated repeatedly. For anything more complex, consider proper language implementation tools.

---

*Based on concepts from "Design Patterns: Elements of Reusable Object-Oriented Software" by Gamma, Helm, Johnson, and Vlissides (Gang of Four), 1994.*
