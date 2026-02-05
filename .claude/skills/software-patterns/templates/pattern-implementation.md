# Pattern Implementation Guide

## How to Generate Implementations

When asked to implement a pattern, follow this process:

### 1. Identify the Pattern
- Load the pattern definition from the appropriate skill
- Understand: intent, participants, collaborations
- Check motivating problem and applicability
- Review consequences and trade-offs

### 2. Adapt to Language
- Use language idioms (see language-idioms.md)
- Apply naming conventions (camelCase vs snake_case)
- Use appropriate type system features
- Leverage language-specific abstractions
- Consider memory model (GC vs manual vs ownership)

### 3. Implementation Template

```
PATTERN: [Name]
LANGUAGE: [Target language]
SCENARIO: [Brief description]

// Core participants
[List each participant with role]

// Implementation
[Provide idiomatic code]

// Usage example
[Show typical usage]

// Trade-offs in this implementation
[Note any compromises or variations]
```

### 4. Common Variations

#### Classical vs Modern
- **Classical**: Direct GoF translation
- **Modern**: Language-native alternatives (e.g., functions over Strategy objects)

#### Interface-based vs Abstract Class
- **Interface**: When no shared implementation, pure contract
- **Abstract**: When shared behavior, partial implementation
- **Trait/Mixin**: When multiple inheritance of behavior needed

#### Functional Alternatives
- **Strategy** → First-class functions
- **Command** → Closures with captured state
- **Observer** → Reactive streams (RxJS, Reactor)
- **Template Method** → Higher-order functions
- **Visitor** → Pattern matching (Rust, modern Java)

#### Framework Integrations
- **Spring/CDI**: Dependency injection replaces many creational patterns
- **React/Vue**: Observer becomes state management
- **ASP.NET**: Middleware chain = Chain of Responsibility
- **Express**: Router = Chain + Command

### 5. Testing Guidance

#### Unit Test Patterns
```
// Test the abstraction, not the pattern
test('strategy selects correct algorithm', () => {
  const context = new Context(new ConcreteStrategyA());
  expect(context.execute()).toBe(expectedResultForA);
});

// Test pattern invariants
test('singleton returns same instance', () => {
  expect(Singleton.getInstance()).toBe(Singleton.getInstance());
});

// Test substitutability
test('all factories produce compatible products', () => {
  [factoryA, factoryB].forEach(factory => {
    const product = factory.createProduct();
    expect(product).toHaveMethod('use');
  });
});
```

#### Mock Strategies
- **Factory**: Mock the factory, not the products
- **Strategy**: Inject test strategy implementation
- **Observer**: Use test observer that records notifications
- **Proxy**: Mock the real subject behind the proxy
- **Adapter**: Mock the adaptee, test the adapter

#### Integration Points
- Test boundaries where patterns compose
- Verify lifecycle management (creation, cleanup)
- Check error handling across pattern boundaries
- Validate thread safety if concurrent

### 6. Anti-Patterns to Avoid

| Pattern | Common Mistake | Better Approach |
|---------|----------------|-----------------|
| Singleton | Global mutable state | Dependency injection |
| Factory | God factory creating everything | Multiple focused factories |
| Strategy | One strategy, defeating purpose | Use simple polymorphism |
| Observer | Memory leaks (unsubscribe) | Weak references or explicit cleanup |
| Decorator | Too many layers (n wrappers) | Composite decorator |
| Template Method | Too granular (dozens of hooks) | Fewer, more meaningful hooks |

### 7. Pattern Composition

#### Common Combinations
- **Abstract Factory + Factory Method**: Family factories use factory methods
- **Builder + Composite**: Build complex composite structures
- **Strategy + Template Method**: Strategies use template method
- **Decorator + Strategy**: Decorated strategies
- **Chain of Responsibility + Command**: Command chain
- **Observer + Mediator**: Mediator notifies observers
- **Prototype + Factory**: Factory returns clones
- **Memento + Command**: Undoable commands store mementos

### 8. Refactoring to Patterns

#### Smells that Suggest Patterns

**Long parameter lists** → Builder, Parameter Object
**Switch on type code** → Strategy, State, Polymorphism
**Parallel inheritance hierarchies** → Bridge
**Refused bequest** → Adapter, Bridge
**Data clumps** → Composite, Value Object
**Message chains** → Facade, Law of Demeter
**Feature envy** → Strategy, Visitor

#### Incremental Refactoring
1. Identify the smell
2. Extract interface/abstraction
3. Create first concrete implementation
4. Replace original code with pattern structure
5. Add variations as new implementations
6. Remove duplication

### 9. Documentation Template

```markdown
## Pattern: [Name]

**Intent**: [One sentence]

**Implementation Notes**:
- [Language-specific considerations]
- [Trade-offs made]
- [Variations from classical]

**Usage**:
```[language]
// Show typical usage
```

**Testing**:
- [Key test scenarios]
- [Mock strategies]

**Related Patterns**:
- [Patterns that work well with this one]
```

### 10. Checklist

Before delivering a pattern implementation:

- [ ] Intent clearly stated
- [ ] Participants identified with roles
- [ ] Idiomatic code for target language
- [ ] Usage example provided
- [ ] Trade-offs documented
- [ ] Test guidance included
- [ ] Common variations noted
- [ ] Anti-patterns warned against
- [ ] Works with existing codebase style
- [ ] Complexity justified by need
