# Pattern Cheatsheet

## Creational Patterns

| Pattern | One-liner | When | Avoid When |
|---------|-----------|------|------------|
| **Factory Method** | Defer instantiation to subclasses | Need to instantiate, don't know which class | Simple construction, use constructor |
| **Abstract Factory** | Create families of related objects | Multiple related objects needed together | Only one product family |
| **Builder** | Construct complex objects step-by-step | Many optional params, complex construction | Few parameters, use constructor |
| **Prototype** | Clone existing objects | Creating is expensive, need copies | Simple construction cheaper |
| **Singleton** | Single global instance | Exactly one instance needed (database connection pool, config) | Use sparingly - prefer DI |

### Quick Decision: Creational
- **Too many constructor params?** → Builder
- **Don't know which class to instantiate?** → Factory Method
- **Need related objects together?** → Abstract Factory
- **Construction is expensive?** → Prototype (clone instead)
- **Exactly one instance?** → Singleton (but consider DI instead)

---

## Structural Patterns

| Pattern | One-liner | When | Avoid When |
|---------|-----------|------|------------|
| **Adapter** | Convert interface A to interface B | Integrate incompatible interfaces | Interfaces already compatible |
| **Bridge** | Separate abstraction from implementation | Abstraction and implementation vary independently | No variation on either axis |
| **Composite** | Tree structure of objects | Part-whole hierarchies, uniform treatment | Not naturally hierarchical |
| **Decorator** | Add responsibilities dynamically | Add features without subclassing | Static responsibilities, use inheritance |
| **Facade** | Simplified interface to subsystem | Simplify complex subsystem | Subsystem already simple |
| **Flyweight** | Share fine-grained objects | Many similar objects, memory-constrained | Objects have unique state |
| **Proxy** | Surrogate controls access | Lazy loading, access control, remote objects | No need for access control or indirection |

### Quick Decision: Structural
- **Need to work with legacy code?** → Adapter
- **Want to add features at runtime?** → Decorator
- **Have part-whole hierarchy?** → Composite
- **Subsystem too complex?** → Facade
- **Abstraction + implementation both vary?** → Bridge
- **Too many similar objects?** → Flyweight (share common state)
- **Need lazy loading or access control?** → Proxy

---

## Behavioral Patterns

| Pattern | One-liner | When | Avoid When |
|---------|-----------|------|------------|
| **Chain of Responsibility** | Pass request along chain until handled | Multiple handlers, don't know which handles | Only one handler |
| **Command** | Encapsulate request as object | Parameterize actions, queue requests, undo | Simple callbacks sufficient |
| **Interpreter** | Define grammar and interpret sentences | Need to evaluate language/expressions | Complex grammar, use parser generator |
| **Iterator** | Access elements sequentially | Need to traverse collection without exposing internals | Language has built-in iteration |
| **Mediator** | Centralize complex communications | Many objects interact, coupling is high | Simple interactions |
| **Memento** | Capture and restore object state | Undo/redo, snapshots | State is simple, use copy |
| **Observer** | Notify dependents of state changes | One-to-many dependency, loose coupling | Tight coupling acceptable |
| **State** | Alter behavior when state changes | Behavior depends on state, many state transitions | Few states, use switch |
| **Strategy** | Encapsulate interchangeable algorithms | Multiple algorithms for same task | Only one algorithm |
| **Template Method** | Define algorithm skeleton, defer steps | Algorithm structure fixed, steps vary | No common structure |
| **Visitor** | Add operations without modifying classes | Many operations on stable class hierarchy | Class hierarchy changes often |

### Quick Decision: Behavioral

#### **Algorithm/Behavior Variations**
- **Varying algorithms for same task?** → Strategy
- **Varying behavior by state?** → State
- **Algorithm structure fixed, steps vary?** → Template Method

#### **Communication/Coordination**
- **Need to notify multiple objects?** → Observer
- **Complex object interactions?** → Mediator
- **Request passes through handlers?** → Chain of Responsibility

#### **Operations on Objects**
- **Need to traverse collection?** → Iterator
- **Add operations without modifying classes?** → Visitor

#### **Request Handling**
- **Need to queue, log, or undo requests?** → Command
- **Multiple handlers, don't know which?** → Chain of Responsibility

#### **State Management**
- **Need undo/redo?** → Command + Memento
- **Capture/restore state?** → Memento
- **Behavior varies by state?** → State

#### **Language/Expression Evaluation**
- **Interpret expressions?** → Interpreter (or use parser library)

---

## Pattern Combinations (Common)

| Combination | Purpose | Example |
|-------------|---------|---------|
| **Abstract Factory + Factory Method** | Families use factory methods | UI toolkit creates platform-specific widgets |
| **Builder + Composite** | Build complex structures | Document builder creates composite doc tree |
| **Strategy + Template Method** | Strategies use template method | Sorting algorithms with fixed logging steps |
| **Decorator + Strategy** | Decorated strategies | Compression strategies with logging decorator |
| **Chain + Command** | Command chain | Request pipeline, middleware |
| **Observer + Mediator** | Mediator notifies observers | Chat room (mediator) notifies users (observers) |
| **Prototype + Factory** | Factory returns clones | Product catalog clones prototypes |
| **Memento + Command** | Undoable commands | Text editor undo/redo |
| **State + Strategy** | State delegates to strategy | Game character state uses different AI strategies |
| **Composite + Visitor** | Operations on tree | Render operation on UI tree |
| **Facade + Singleton** | Single simplified interface | Single point of access to subsystem |
| **Proxy + Factory** | Factory creates proxies | ORM factory returns lazy-loading proxies |

---

## Pattern Selection by Symptom

### Code Smells → Pattern Fix

| Smell | Pattern Solution |
|-------|------------------|
| **Long parameter lists** | Builder, Parameter Object |
| **Switch on type code** | Strategy, State, Polymorphism |
| **Parallel inheritance hierarchies** | Bridge |
| **Refused bequest** (subclass doesn't use inherited methods) | Adapter, Bridge |
| **Data clumps** | Composite, Value Object |
| **Message chains** (`a.getB().getC().getD()`) | Facade, Law of Demeter |
| **Feature envy** (method uses another class more than its own) | Strategy, Visitor |
| **Shotgun surgery** (change requires many small changes) | Mediator, Facade |
| **Divergent change** (class changes for many reasons) | Strategy, State, Extract Class |
| **Large class** | Facade, Composite, Extract Class |

### Goals → Pattern Solution

| Goal | Pattern |
|------|---------|
| **Add features without modifying code** | Decorator, Strategy, Visitor |
| **Reduce coupling** | Observer, Mediator, Facade |
| **Vary algorithm** | Strategy, Template Method |
| **Vary object creation** | Factory Method, Abstract Factory, Builder |
| **Control access** | Proxy |
| **Simplify complex interface** | Facade |
| **Undo/redo** | Command + Memento |
| **Traverse collection** | Iterator |
| **Change behavior by state** | State |
| **Integrate incompatible code** | Adapter |
| **Part-whole hierarchy** | Composite |
| **Share memory** | Flyweight |

---

## Quick Reference by Use Case

### **Need to create objects?**
1. Don't know which class? → **Factory Method**
2. Related objects together? → **Abstract Factory**
3. Many constructor params? → **Builder**
4. Expensive creation? → **Prototype**
5. Exactly one instance? → **Singleton** (or DI)

### **Need to structure code?**
1. Incompatible interfaces? → **Adapter**
2. Add features dynamically? → **Decorator**
3. Tree of objects? → **Composite**
4. Simplify subsystem? → **Facade**
5. Abstraction + implementation vary? → **Bridge**
6. Many similar objects? → **Flyweight**
7. Control access? → **Proxy**

### **Need to handle behavior/algorithms?**
1. Vary algorithms? → **Strategy**
2. Vary by state? → **State**
3. Fixed steps, vary details? → **Template Method**

### **Need to communicate between objects?**
1. Notify many objects? → **Observer**
2. Complex interactions? → **Mediator**
3. Chain of handlers? → **Chain of Responsibility**

### **Need to work with collections/structures?**
1. Traverse collection? → **Iterator**
2. Operations on stable hierarchy? → **Visitor**
3. Tree structure? → **Composite**

### **Need request handling?**
1. Parameterize/queue/undo? → **Command**
2. Multiple handlers? → **Chain of Responsibility**

### **Need state management?**
1. Undo/redo? → **Command** + **Memento**
2. Snapshot/restore? → **Memento**
3. Behavior depends on state? → **State**

---

## Pattern Relationships

### Patterns that Work Well Together

```
Factory Method ←→ Template Method (factory method is template method for creation)
Abstract Factory ←→ Singleton (factories often singletons)
Builder ←→ Composite (build composite structures)
Prototype ←→ Factory (factory clones prototypes)

Adapter ←→ Facade (facade uses adapters internally)
Decorator ←→ Composite (decorators can be composed)
Proxy ←→ Decorator (similar structure, different intent)

Strategy ←→ State (similar structure, state self-transitions)
Command ←→ Memento (commands store mementos for undo)
Observer ←→ Mediator (mediator can notify observers)
Chain ←→ Composite (composite can be chain node)
Iterator ←→ Composite (iterate composite structures)
Visitor ←→ Composite (visit composite structures)
```

### Patterns that Are Alternatives

```
Strategy vs State: Strategy chosen by client, State self-transitions
Template Method vs Strategy: Template uses inheritance, Strategy uses composition
Factory Method vs Abstract Factory: Single vs family of products
Decorator vs Strategy: Decorator stacks, Strategy picks one
Proxy vs Decorator: Proxy controls access, Decorator adds features
```

---

## Anti-Patterns to Avoid

| Pattern | Common Misuse | Better Approach |
|---------|---------------|-----------------|
| **Singleton** | Global mutable state | Dependency injection |
| **Singleton** | Testing nightmare | Pass dependencies explicitly |
| **Factory** | God factory creating everything | Multiple focused factories |
| **Strategy** | Only one strategy (pointless) | Just use the class directly |
| **Template Method** | Too many hooks (dozens) | Fewer, meaningful hooks or Strategy |
| **Decorator** | Too many layers (10+ wrappers) | Composite decorator or rethink |
| **Observer** | Memory leaks (no unsubscribe) | Weak references or explicit cleanup |
| **Composite** | Not naturally hierarchical | Don't force it |
| **Visitor** | Volatile class hierarchy | Use Strategy or simple polymorphism |
| **Chain** | No handler catches request | Always have default handler |
| **Mediator** | God object mediator | Keep mediator focused |
| **Memento** | Exposing internal state | Keep memento opaque |

---

## Modern Alternatives

| Classical Pattern | Modern Alternative | When to Use Modern |
|-------------------|--------------------|--------------------|
| **Strategy** | First-class functions/lambdas | Single method, stateless |
| **Command** | Closures with captured state | Simple operations, no undo |
| **Observer** | Reactive streams (RxJS, Reactor) | Complex event flows |
| **Template Method** | Higher-order functions | Functional language style |
| **Factory** | Dependency injection container | Framework provides DI |
| **Singleton** | Module scope, DI | No global mutable state |
| **Iterator** | Language built-in iteration | Python generators, JS iterators |
| **Visitor** | Pattern matching (Rust, modern Java) | Exhaustive matching |
| **State** | Enum with data (Rust, Kotlin) | Closed state set |
| **Builder** | Named/optional parameters | Language supports it |

---

## Complexity Guide

### Lightweight (Start Here)
- **Strategy** (just use functions in many languages)
- **Factory Method** (static factory methods)
- **Adapter** (thin wrapper)
- **Facade** (convenience methods)
- **Observer** (simple pub-sub)

### Medium Weight (When Needed)
- **Builder** (for complex construction)
- **Decorator** (dynamic features)
- **Command** (for undo/queuing)
- **State** (explicit state machine)
- **Template Method** (reusable algorithm structure)

### Heavyweight (Use Sparingly)
- **Abstract Factory** (whole product families)
- **Visitor** (operations on stable hierarchy)
- **Interpreter** (if you need a DSL)
- **Flyweight** (only for serious memory issues)
- **Mediator** (can become god object)

---

## One-Sentence Summaries

1. **Factory Method**: Let subclasses decide which class to instantiate
2. **Abstract Factory**: Families of related objects without specifying concrete classes
3. **Builder**: Separate construction from representation for complex objects
4. **Prototype**: Create new objects by cloning existing ones
5. **Singleton**: Ensure one instance and global access point
6. **Adapter**: Convert one interface to another
7. **Bridge**: Decouple abstraction from implementation
8. **Composite**: Part-whole hierarchy, treat individuals and compositions uniformly
9. **Decorator**: Add responsibilities to objects dynamically
10. **Facade**: Unified interface to subsystem
11. **Flyweight**: Share fine-grained objects efficiently
12. **Proxy**: Surrogate controls access to another object
13. **Chain of Responsibility**: Pass request along chain until handled
14. **Command**: Encapsulate request as object
15. **Interpreter**: Define grammar and interpreter for language
16. **Iterator**: Access elements sequentially without exposing representation
17. **Mediator**: Centralize complex communications between objects
18. **Memento**: Capture and restore object state without violating encapsulation
19. **Observer**: Define one-to-many dependency for automatic notification
20. **State**: Object behavior changes when internal state changes
21. **Strategy**: Define family of algorithms, make them interchangeable
22. **Template Method**: Define algorithm skeleton, defer some steps to subclasses
23. **Visitor**: Add operations to classes without changing them

---

## Memory Aids

### By Letter (Mnemonic)

**Creational: "FABPS"**
- F: Factory Method
- A: Abstract Factory
- B: Builder
- P: Prototype
- S: Singleton

**Structural: "ABC DFFP"**
- A: Adapter
- B: Bridge
- C: Composite
- D: Decorator
- F: Facade
- F: Flyweight
- P: Proxy

**Behavioral: "C CII MMOSS TTTV"** (tricky to remember - use categories instead)
- Communication: Chain, Command, Mediator, Observer
- Algorithms: Strategy, State, Template Method
- Traversal: Iterator, Visitor
- Other: Interpreter, Memento

### By Category (Easier)

**Object Creation**: Factory Method, Abstract Factory, Builder, Prototype, Singleton

**Interface Adaptation**: Adapter, Facade, Proxy

**Structure**: Composite, Decorator, Bridge, Flyweight

**Algorithm Behavior**: Strategy, State, Template Method

**Communication**: Observer, Mediator, Chain of Responsibility

**Request Handling**: Command, Chain of Responsibility

**State/History**: Memento, State

**Traversal/Operations**: Iterator, Visitor

**Language**: Interpreter

---

## When NOT to Use Patterns

**Don't use patterns when:**
1. The problem is simple (don't over-engineer)
2. The language has a built-in feature (e.g., Python decorators vs Decorator pattern)
3. You're adding them "just in case" (YAGNI principle)
4. The pattern increases complexity without clear benefit
5. You're pattern-matching without understanding intent

**Remember**: Patterns are solutions to recurring problems. No problem? No pattern.
