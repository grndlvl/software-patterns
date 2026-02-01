# Domain Languages

## Core Principle

> "Don't program in the specification language. Instead, write a mini-language that lets you express the problem more directly." - The Pragmatic Programmer

Domain-Specific Languages (DSLs) allow you to work closer to the problem domain rather than the implementation domain. By creating specialized languages for specific tasks, you make code more declarative, maintainable, and understandable to domain experts.

## Internal vs External DSLs

### Internal DSLs
Languages built within a host programming language, leveraging its syntax and features.

**Characteristics:**
- Use host language's parser and compiler
- Look like native code but express domain concepts
- Easier to implement and maintain
- Share tooling with host language

**Example:**
```pseudocode
// Configuration DSL embedded in host language
database.configure do
  set connection_pool to 10
  set timeout to 30 seconds
  enable query_caching
  use replica "db-slave-01" for read_operations
end
```

### External DSLs
Standalone languages with custom syntax, requiring their own parsers.

**Characteristics:**
- Complete control over syntax
- Can be highly specialized
- Requires parser/interpreter development
- May need custom tooling

**Example:**
```pseudocode
// Custom configuration language
DATABASE_CONFIG {
  connection_pool: 10
  timeout: 30s
  query_cache: enabled
  read_replica: db-slave-01
}
```

## When to Use Domain Languages

### Strong Indicators

| Scenario | Why DSL Helps |
|----------|---------------|
| **Repeated patterns** | Complex logic appears in many places with slight variations |
| **Domain expert involvement** | Non-programmers need to understand or modify rules |
| **Configuration complexity** | Settings have intricate relationships and validation rules |
| **Business rule volatility** | Rules change frequently and independently of core logic |
| **Data transformation pipelines** | Multi-step processes with clear input/output contracts |

### Anti-Indicators

| Scenario | Why to Avoid |
|----------|--------------|
| **One-off tasks** | Overhead exceeds benefits for single-use code |
| **Simple mappings** | JSON or YAML configuration suffices |
| **Rapidly changing requirements** | DSL design can't stabilize |
| **Small team without expertise** | Maintenance burden too high |

## Building Domain Languages

### Mini-Languages (Executable)

Small languages designed to perform specific computational tasks.

**Pattern Matching Example:**
```pseudocode
// Email validation mini-language
RULE email_validator:
  PATTERN: one_or_more(alphanumeric or [. _ -])
          "@"
          one_or_more(alphanumeric or [. -])
          "."
          between(2, 6, alphabetic)

  VALIDATE:
    not starts_with(".")
    not ends_with(".")
    not contains("..")
END RULE
```

**Workflow Definition Example:**
```pseudocode
// Order processing workflow
WORKFLOW order_fulfillment:
  STEP validate_payment:
    CHECK payment_method is valid
    CHECK funds_available >= order_total
    ON_FAIL: notify customer, halt

  STEP reserve_inventory:
    FOR_EACH item IN order_items:
      LOCK inventory_quantity FOR item
    ON_FAIL: release_locks, rollback, halt

  STEP ship_order:
    ASSIGN warehouse = nearest_to(customer_address)
    CREATE shipping_label
    NOTIFY warehouse_system

  COMPLETION:
    SEND confirmation_email TO customer
    LOG analytics_event "order_completed"
END WORKFLOW
```

### Data Languages (Declarative)

Languages that describe structures, relationships, or configurations.

**Schema Definition Example:**
```pseudocode
// Data validation schema language
SCHEMA user_profile:
  FIELD username:
    TYPE: string
    CONSTRAINTS:
      length BETWEEN 3 AND 20
      matches PATTERN "^[a-zA-Z0-9_]+$"
      UNIQUE IN database

  FIELD email:
    TYPE: email_address
    REQUIRED
    UNIQUE IN database

  FIELD age:
    TYPE: integer
    OPTIONAL
    CONSTRAINTS:
      MINIMUM 13
      MAXIMUM 120

  FIELD preferences:
    TYPE: nested_object
    SCHEMA:
      FIELD notifications:
        TYPE: boolean
        DEFAULT: true
      FIELD theme:
        TYPE: enum["light", "dark", "auto"]
        DEFAULT: "auto"
END SCHEMA
```

**Access Control Example:**
```pseudocode
// Permission definition language
PERMISSIONS:
  ROLE administrator:
    CAN perform ANY action ON ANY resource

  ROLE editor:
    CAN create, read, update ON documents
    CAN read ON users
    WHERE user.department EQUALS current_user.department

  ROLE viewer:
    CAN read ON documents
    WHERE document.published EQUALS true
      OR document.author EQUALS current_user

  ROLE guest:
    CAN read ON documents
    WHERE document.visibility EQUALS "public"
END PERMISSIONS
```

## Implementation Strategies

### Lexical Analysis Approach
```pseudocode
// Simple tokenizer for DSL
FUNCTION tokenize(input_text):
  tokens = empty_list
  position = 0

  WHILE position < length(input_text):
    character = input_text[position]

    IF character matches whitespace:
      position = position + 1
      CONTINUE

    IF character matches letter:
      identifier = extract_identifier(input_text, position)
      tokens.add(TOKEN("IDENTIFIER", identifier))
      position = position + length(identifier)

    IF character matches digit:
      number = extract_number(input_text, position)
      tokens.add(TOKEN("NUMBER", number))
      position = position + length(number)

    IF character matches operator:
      tokens.add(TOKEN("OPERATOR", character))
      position = position + 1

  RETURN tokens
END FUNCTION
```

### Parser Pattern
```pseudocode
// Recursive descent parser example
FUNCTION parse_expression(tokens):
  left = parse_term(tokens)

  WHILE current_token() IN ["+", "-"]:
    operator = consume_token()
    right = parse_term(tokens)
    left = create_binary_expression(operator, left, right)

  RETURN left
END FUNCTION

FUNCTION parse_term(tokens):
  left = parse_factor(tokens)

  WHILE current_token() IN ["*", "/"]:
    operator = consume_token()
    right = parse_factor(tokens)
    left = create_binary_expression(operator, left, right)

  RETURN left
END FUNCTION

FUNCTION parse_factor(tokens):
  token = current_token()

  IF token.type EQUALS "NUMBER":
    consume_token()
    RETURN create_number_node(token.value)

  IF token.type EQUALS "IDENTIFIER":
    consume_token()
    RETURN create_variable_node(token.value)

  IF token.value EQUALS "(":
    consume_token()  // consume "("
    expression = parse_expression(tokens)
    expect_token(")")  // consume ")"
    RETURN expression

  RAISE syntax_error("Unexpected token")
END FUNCTION
```

## Trade-offs and Considerations

### Benefits

| Benefit | Impact |
|---------|--------|
| **Clarity** | Domain concepts expressed directly without translation layer |
| **Validation** | DSL enforces domain rules at parse time |
| **Productivity** | Experts can modify behavior without touching implementation |
| **Testability** | DSL scripts can be tested independently |
| **Evolution** | Domain logic changes without code recompilation |

### Costs

| Cost | Mitigation Strategy |
|------|---------------------|
| **Learning curve** | Comprehensive documentation, clear error messages |
| **Tooling investment** | Start with internal DSL, leverage existing tools |
| **Debugging difficulty** | Generate source maps, provide runtime introspection |
| **Maintenance burden** | Keep grammar simple, version the language spec |
| **Performance overhead** | Cache parsed results, compile to native code if needed |

### Design Guidelines

1. **Start Small**: Begin with limited scope, expand based on real needs
2. **Parse, Don't Validate**: Use type systems and structure to prevent invalid states
3. **Fail Fast**: Report errors at parse time, not runtime
4. **Optimize for Reading**: DSL will be read 10x more than written
5. **Document Extensively**: Include examples for every construct
6. **Version Carefully**: DSL changes are breaking changes for users

## Complexity Ladder

```pseudocode
// Level 1: Simple data file (not really a DSL)
config_file = {
  "timeout": 30,
  "retries": 3
}

// Level 2: Structured data with conventions
CONFIGURATION:
  timeout: 30 seconds
  retry_policy: exponential_backoff(3 attempts)

// Level 3: Mini-language with control flow
WHEN request_timeout:
  RETRY with exponential_backoff
    initial_delay: 1 second
    max_attempts: 3
    max_delay: 10 seconds
  IF all_retries_exhausted:
    NOTIFY operations_team
    RETURN error_to_client

// Level 4: Full DSL with abstractions
POLICY request_resilience:
  DEFINE retry_strategy AS exponential_backoff:
    base_delay: 1s
    multiplier: 2
    max_attempts: 3

  ON timeout_error:
    APPLY retry_strategy
    LOG "Request timeout, retrying"

  ON max_retries_exceeded:
    ESCALATE TO operations_team WITH context
    RESPOND TO client WITH friendly_error_message
END POLICY
```

## Real-World Applications

### Build Systems
```pseudocode
// Declarative build DSL
BUILD target "web-app":
  SOURCES: find_files("src/**/*.ts")

  STEP transpile:
    TOOL: typescript_compiler
    OPTIONS: strict_mode, source_maps
    OUTPUT: "dist/js"

  STEP bundle:
    TOOL: module_bundler
    INPUT: transpile.output
    OPTIONS: minify, tree_shake
    OUTPUT: "dist/bundle.js"

  STEP optimize:
    TOOL: asset_optimizer
    INPUT: bundle.output, find_files("assets/**")
    OUTPUT: "dist/optimized"

  WATCH_FILES: "src/**"
  ON_CHANGE: rebuild_incrementally
END BUILD
```

### Testing DSL
```pseudocode
// Behavior specification language
SCENARIO "User registration with valid data":
  GIVEN user visits registration_page
  AND user is not logged_in

  WHEN user enters:
    username: "alice_smith"
    email: "alice@example.com"
    password: "SecurePass123!"
  AND user clicks submit_button

  THEN user should be redirected_to welcome_page
  AND user should receive confirmation_email
  AND database should contain user_record WITH:
    username: "alice_smith"
    email: "alice@example.com"
    verified: false
END SCENARIO
```

## Summary: DSL Decision Matrix

| Factor | Score +1 if True | Your Score |
|--------|------------------|------------|
| Task repeats 10+ times in codebase | ✓ or ✗ | ___ |
| Non-programmers need to understand/modify | ✓ or ✗ | ___ |
| Domain has stable, well-understood concepts | ✓ or ✗ | ___ |
| Changes happen frequently | ✓ or ✗ | ___ |
| Team has language design experience | ✓ or ✗ | ___ |
| Host language is verbose for this domain | ✓ or ✗ | ___ |
| Cost of errors is high | ✓ or ✗ | ___ |

**Scoring:**
- **0-2**: Stick with general-purpose language or configuration files
- **3-4**: Consider internal DSL
- **5-7**: Strong candidate for external DSL

## Key Takeaways

1. **Start with data languages** - easier to build, lower risk
2. **Internal DSLs first** - leverage existing tooling and expertise
3. **Design for humans** - clarity trumps cleverness
4. **Parse-time validation** - catch errors before execution
5. **Document ruthlessly** - DSL without docs is a liability
6. **Version explicitly** - treat DSL as a contract with users
7. **Measure success** - does it reduce errors and increase velocity?

---

*Based on concepts from "The Pragmatic Programmer" by David Thomas and Andrew Hunt.*
