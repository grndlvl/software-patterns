# Interpreters

## Metalinguistic Abstraction

Metalinguistic abstraction means designing a language to directly express solutions to problems. Rather than writing programs that solve problems in an existing language, we design new languages that make problem expression natural and direct.

**Key Benefits:**
- Express solutions at the problem's natural level of abstraction
- Implement domain-specific languages (DSLs)
- Extend existing languages with new constructs
- Create layers of abstraction that match problem domain

**Core Insight:** A program written in a new language is just data to an interpreter. The interpreter implements the new language's semantics by evaluating that data.

---

## The Eval-Apply Cycle

The fundamental mechanism of any interpreter consists of two mutually recursive functions:

### EVAL
Evaluates an expression in an environment to produce a value.

```pseudocode
function EVAL(expression, environment):
    if isPrimitive(expression):
        return getValueInEnvironment(expression, environment)
    else if isSpecialForm(expression):
        return evaluateSpecialForm(expression, environment)
    else:
        // It's a function application
        function = EVAL(car(expression), environment)
        arguments = EVAL_LIST(cdr(expression), environment)
        return APPLY(function, arguments)
```

### APPLY
Applies a function (procedure) to a list of argument values.

```pseudocode
function APPLY(procedure, argumentValues):
    if isPrimitive(procedure):
        return executePrimitive(procedure, argumentValues)
    else if isCompoundProcedure(procedure):
        newEnv = createEnv(
            parameterNames = procedure.parameters,
            parameterValues = argumentValues,
            parentEnv = procedure.closureEnv
        )
        return evalSequence(procedure.body, newEnv)
```

**Cycle Pattern:**
1. EVAL determines what expression means
2. If it's an application, EVAL recursively evaluates subexpressions
3. APPLY executes the function with evaluated arguments
4. Primitive procedures delegate to underlying system
5. Compound procedures create new environments and EVAL their bodies

---

## Representing Expressions

Expressions are represented as data structures. A program is interpreted by recursively analyzing its structure.

### 1. Self-Evaluating Expressions

Evaluate to themselves (numbers, strings, booleans, etc.)

```pseudocode
function isSelfEvaluating(expression):
    return isNumber(expression) or
           isString(expression) or
           isBoolean(expression)

function evalSelfEvaluating(expression, environment):
    return expression
```

**Example:** `42`, `"hello"`, `true`

### 2. Variables

Symbols that refer to bound values in the environment.

```pseudocode
function isVariable(expression):
    return isSymbol(expression)

function evalVariable(expression, environment):
    binding = lookupVariable(expression, environment)
    if binding == null:
        throw UnboundVariableError(expression)
    return binding.value
```

**Example:** `x`, `sum`, `current-input-port`

### 3. Quoted Expressions

Treat a piece of data as a literal value, not code to evaluate.

```pseudocode
function isQuoted(expression):
    return car(expression) == symbol("quote")

function evalQuoted(expression, environment):
    // (quote x) returns x literally, unevaluated
    return cadr(expression)

// Shorthand: 'x is syntax sugar for (quote x)
```

**Example:** `(quote (+ 1 2))` returns the list `(+ 1 2)`, not 3

### 4. Assignments

Bind a symbol to a value in the current environment.

```pseudocode
function isAssignment(expression):
    return car(expression) == symbol("set!")

function evalAssignment(expression, environment):
    variable = cadr(expression)
    newValue = EVAL(caddr(expression), environment)
    setVariableValue(variable, newValue, environment)
    return symbol("ok")  // Return unspecified value
```

**Example:** `(set! x 5)` - rebind x to 5 in current environment

### 5. Definitions

Create new bindings in the environment.

```pseudocode
function isDefinition(expression):
    return car(expression) == symbol("define")

function evalDefinition(expression, environment):
    variable = cadr(expression)

    if isSymbol(variable):
        // (define x 10)
        value = EVAL(caddr(expression), environment)
        defineVariable(variable, value, environment)
    else:
        // (define (f x y) ...) - syntactic sugar for lambda
        // Transform: (define (f x y) body)
        // Into:      (define f (lambda (x y) body))
        functionName = car(variable)
        parameters = cdr(variable)
        body = cddr(expression)
        procedure = makeCompound(parameters, body, environment)
        defineVariable(functionName, procedure, environment)

    return symbol("ok")
```

**Examples:**
- `(define x 10)` - bind x to 10
- `(define (square x) (* x x))` - define a function

### 6. Conditionals (if)

Evaluate test, then evaluate appropriate consequent or alternative.

```pseudocode
function isIf(expression):
    return car(expression) == symbol("if")

function evalIf(expression, environment):
    test = cadr(expression)
    consequent = caddr(expression)
    alternative = cadddr(expression)

    testValue = EVAL(test, environment)

    if isTruthy(testValue):
        return EVAL(consequent, environment)
    else:
        return EVAL(alternative, environment)

function isTruthy(value):
    // Only false is falsy; everything else is truthy
    return value != false
```

**Example:** `(if (> x 0) (positive x) (non-positive x))`

### 7. Lambda (Function Definition)

Create a procedure object capturing the parameters, body, and closure environment.

```pseudocode
function isLambda(expression):
    return car(expression) == symbol("lambda")

function evalLambda(expression, environment):
    parameters = cadr(expression)
    body = cddr(expression)

    return makeCompound(
        parameters = parameters,
        body = body,
        closureEnv = environment  // Capture environment!
    )
```

**Example:** `(lambda (x y) (+ (* x x) (* y y)))`

### 8. Begin (Sequencing)

Evaluate expressions in sequence, return value of last.

```pseudocode
function isBegin(expression):
    return car(expression) == symbol("begin")

function evalBegin(expression, environment):
    expressions = cdr(expression)
    result = null

    for expr in expressions:
        result = EVAL(expr, environment)

    return result
```

**Example:** `(begin (set! x 1) (set! y 2) (+ x y))`

### 9. Application (Function Call)

Evaluate operator and operands, then apply.

```pseudocode
function isApplication(expression):
    return isPair(expression) and not isSpecialForm(expression)

function evalApplication(expression, environment):
    operator = EVAL(car(expression), environment)
    operands = cdr(expression)
    argumentValues = evalList(operands, environment)
    return APPLY(operator, argumentValues)

function evalList(expressions, environment):
    if isNull(expressions):
        return null
    else:
        first = EVAL(car(expressions), environment)
        rest = evalList(cdr(expressions), environment)
        return cons(first, rest)
```

**Example:** `(+ 1 2)`, `(square 5)`, `((lambda (x) (* x x)) 4)`

---

## Environment Operations

The environment is the context where variables are bound to values. It's typically implemented as a chain of frames.

### Structure

```pseudocode
class Frame:
    variables: list of symbols
    values: list of corresponding values

class Environment:
    frame: Frame
    enclosingEnv: Environment (or null)
```

### Key Operations

```pseudocode
function defineVariable(variable, value, environment):
    // Add binding to the frame of environment
    frame = environment.frame
    index = findIndex(frame.variables, variable)

    if index != -1:
        // Update existing binding
        frame.values[index] = value
    else:
        // Add new binding
        frame.variables.append(variable)
        frame.values.append(value)

function setVariableValue(variable, value, environment):
    // Update binding in the frame where variable is defined
    while environment != null:
        frame = environment.frame
        index = findIndex(frame.variables, variable)

        if index != -1:
            // Found the binding
            frame.values[index] = value
            return

        // Search in enclosing environment
        environment = environment.enclosingEnv

    throw UnboundVariableError(variable)

function lookupVariable(variable, environment):
    // Find binding in environment chain
    while environment != null:
        frame = environment.frame
        index = findIndex(frame.variables, variable)

        if index != -1:
            return frame.values[index]

        // Search in enclosing environment
        environment = environment.enclosingEnv

    throw UnboundVariableError(variable)

function createEnvironment(parameterNames, parameterValues, parentEnvironment):
    // Create new environment with parameters as bindings
    if length(parameterNames) != length(parameterValues):
        throw ArityError("Procedure expects " + length(parameterNames) +
                        " arguments, got " + length(parameterValues))

    newFrame = Frame()
    newFrame.variables = parameterNames
    newFrame.values = parameterValues

    return Environment(frame = newFrame, enclosingEnv = parentEnvironment)
```

---

## Pseudocode for Eval and Apply Functions

### Complete EVAL Function

```pseudocode
function EVAL(expression, environment):
    // Self-evaluating
    if isSelfEvaluating(expression):
        return expression

    // Variable lookup
    if isVariable(expression):
        return lookupVariable(expression, environment)

    // Special forms
    if isQuoted(expression):
        return cadr(expression)

    if isAssignment(expression):
        variable = cadr(expression)
        newValue = EVAL(caddr(expression), environment)
        setVariableValue(variable, newValue, environment)
        return symbol("ok")

    if isDefinition(expression):
        return evalDefinition(expression, environment)

    if isIf(expression):
        test = EVAL(cadr(expression), environment)
        if isTruthy(test):
            return EVAL(caddr(expression), environment)
        else:
            return EVAL(cadddr(expression), environment)

    if isLambda(expression):
        parameters = cadr(expression)
        body = cddr(expression)
        return makeCompound(parameters, body, environment)

    if isBegin(expression):
        expressions = cdr(expression)
        result = null
        for expr in expressions:
            result = EVAL(expr, environment)
        return result

    // Application (function call)
    if isApplication(expression):
        function = EVAL(car(expression), environment)
        args = mapEval(cdr(expression), environment)
        return APPLY(function, args)

    throw UnknownExpressionError(expression)

function mapEval(expressions, environment):
    // Evaluate each expression in a list
    if isNull(expressions):
        return null
    else:
        first = EVAL(car(expressions), environment)
        rest = mapEval(cdr(expressions), environment)
        return cons(first, rest)
```

### Complete APPLY Function

```pseudocode
function APPLY(procedure, argumentValues):
    if isPrimitiveFunction(procedure):
        // Built-in procedures like +, -, *, etc.
        return executePrimitive(procedure, argumentValues)

    if isCompoundProcedure(procedure):
        // User-defined procedures
        env = extendEnvironment(
            parameters = procedure.parameters,
            arguments = argumentValues,
            baseEnv = procedure.environment
        )

        body = procedure.body

        // Evaluate the sequence of expressions in the body
        result = null
        for expression in body:
            result = EVAL(expression, env)

        return result

    throw NotAProcedureError(procedure)

function extendEnvironment(parameters, arguments, baseEnv):
    if length(parameters) != length(arguments):
        throw ArityError("Expected " + length(parameters) +
                        " arguments, got " + length(arguments))

    newFrame = Frame()
    newFrame.variables = parameters
    newFrame.values = arguments

    return Environment(frame = newFrame, enclosingEnv = baseEnv)
```

---

## Extending the Evaluator

The evaluator can be extended by:

### 1. Adding Primitive Procedures

```pseudocode
function setupGlobalEnvironment():
    globalEnv = Environment(frame = Frame())

    // Add primitive functions
    definePrimitive("+", builtinAdd, globalEnv)
    definePrimitive("-", builtinSubtract, globalEnv)
    definePrimitive("*", builtinMultiply, globalEnv)
    definePrimitive("/", builtinDivide, globalEnv)
    definePrimitive("=", builtinNumEqual, globalEnv)
    definePrimitive("<", builtinLessThan, globalEnv)
    definePrimitive(">", builtinGreaterThan, globalEnv)
    definePrimitive("car", builtinCar, globalEnv)
    definePrimitive("cdr", builtinCdr, globalEnv)
    definePrimitive("cons", builtinCons, globalEnv)
    definePrimitive("list", builtinList, globalEnv)
    definePrimitive("null?", builtinNull, globalEnv)
    definePrimitive("eq?", builtinEq, globalEnv)

    return globalEnv
```

### 2. Adding Special Forms

Special forms require custom evaluation rules that don't follow normal argument evaluation.

```pseudocode
function isSpecialForm(expression):
    return isQuoted(expression) or
           isAssignment(expression) or
           isDefinition(expression) or
           isIf(expression) or
           isLambda(expression) or
           isBegin(expression) or
           isCustomSpecialForm(expression)

// Example: cond (conditional with multiple clauses)
function isCond(expression):
    return car(expression) == symbol("cond")

function evalCond(expression, environment):
    clauses = cdr(expression)
    return evalCondClauses(clauses, environment)

function evalCondClauses(clauses, environment):
    if isNull(clauses):
        return symbol("false")

    clause = car(clauses)
    test = car(clause)

    if test == symbol("else"):
        // Else clause always succeeds
        consequent = cdr(clause)
        return evalSequence(consequent, environment)
    else:
        testValue = EVAL(test, environment)
        if isTruthy(testValue):
            consequent = cdr(clause)
            return evalSequence(consequent, environment)
        else:
            // Try next clause
            return evalCondClauses(cdr(clauses), environment)
```

### 3. Adding Derived Expressions

Create new forms by transforming them into existing forms.

```pseudocode
// and special form as a derived form
function isAnd(expression):
    return car(expression) == symbol("and")

function transformAnd(expression):
    // (and x y z) => (if x (if y z false) false)
    operands = cdr(expression)
    if isNull(operands):
        return true
    else:
        return cons(symbol("if"),
                   cons(car(operands),
                        cons(transformAnd(cons(symbol("and"), cdr(operands))),
                             cons(false, null))))
```

### 4. Adding Control Structures

Implement new control flow mechanisms.

```pseudocode
// Example: while loop
function isWhile(expression):
    return car(expression) == symbol("while")

function evalWhile(expression, environment):
    test = cadr(expression)
    body = cddr(expression)

    result = null
    while isTruthy(EVAL(test, environment)):
        for expr in body:
            result = EVAL(expr, environment)

    return result
```

---

## Data as Programs Concept

A fundamental insight: **There is no essential difference between data and programs.**

### Programs are Data

```pseudocode
// A program is just a list structure
program = list(symbol("define"),
              symbol("square"),
              list(symbol("lambda"),
                   list(symbol("x")),
                   list(symbol("*"), symbol("x"), symbol("x"))))

// We can manipulate it:
name = car(cdr(program))           // square
parameters = car(cadr(car(cdddr(program))))  // (x)

// Then evaluate it
EVAL(program, globalEnv)
```

### Programs can Generate Programs

```pseudocode
// A macro: code that generates code
function expandTwice(expression):
    // (twice f) => (lambda (x) (f (f x)))
    f = cadr(expression)
    x = symbol("x")

    return list(symbol("lambda"),
               list(x),
               list(f, list(f, x)))

// Usage
program = expandTwice(list(symbol("twice"), symbol("square")))
// Creates: (lambda (x) (square (square x)))
```

### Quote and Eval

Quote and eval enable homoiconicity (code and data have same representation):

```pseudocode
// Using quote to create data that looks like code
data = quote((+ 1 2))

// Using eval to interpret data as code
result = EVAL(data, environment)  // Evaluates (+ 1 2) => 3
```

---

## Summary Table

| Concept | Purpose | Key Mechanism |
|---------|---------|---------------|
| **Metalinguistic Abstraction** | Design languages matching problem domain | Interpreter evaluates domain-specific expressions |
| **EVAL-APPLY Cycle** | Core evaluation loop | EVAL analyzes expressions, APPLY executes procedures |
| **Self-Evaluating** | Numbers, strings, booleans | Return value directly |
| **Variables** | Symbol lookup | Search environment chain |
| **Quoted** | Literal data | Return unevaluated |
| **Assignment** | Modify binding | Update in environment |
| **Definition** | Create new binding | Add to environment |
| **If** | Conditional evaluation | Evaluate test, choose branch |
| **Lambda** | Function abstraction | Create compound procedure |
| **Begin** | Sequencing | Evaluate each, return last |
| **Application** | Function call | Evaluate operator, arguments, apply |
| **Environment** | Variable bindings | Chain of frames |
| **Special Forms** | Non-standard evaluation | Custom eval rules |
| **Derived Forms** | Syntactic sugar | Transform into primitives |
| **Homoiconicity** | Code = Data | Quote enables manipulation |

---

*Based on concepts from "Structure and Interpretation of Computer Programs" by Abelson and Sussman.*
