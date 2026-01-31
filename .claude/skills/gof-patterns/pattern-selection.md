# Design Pattern Selection Guide

A practical guide for choosing the right design pattern for your problem.

## Quick Decision Tree

```
START: What problem are you solving?

├─ Need to create objects?
│  ├─ Complex construction process? → Builder
│  ├─ Only one instance allowed? → Singleton
│  ├─ Create without specifying class? → Factory Method
│  ├─ Families of related objects? → Abstract Factory
│  └─ Copy existing objects? → Prototype
│
├─ Need to add behavior or structure?
│  ├─ Add responsibilities dynamically? → Decorator
│  ├─ Convert one interface to another? → Adapter
│  ├─ Simplify complex subsystem? → Facade
│  ├─ Share objects to save memory? → Flyweight
│  ├─ Separate abstraction from implementation? → Bridge
│  └─ Build complex structure from simple parts? → Composite
│
├─ Need to manage behavior or algorithms?
│  ├─ Notify multiple objects of changes? → Observer
│  ├─ Define family of algorithms? → Strategy
│  ├─ Encapsulate requests as objects? → Command
│  ├─ Traverse collection without exposing structure? → Iterator
│  ├─ Allow undo operations? → Memento
│  ├─ Change behavior when state changes? → State
│  ├─ Coordinate complex interactions? → Mediator
│  ├─ Define skeleton, let subclasses override? → Template Method
│  ├─ Pass request along chain until handled? → Chain of Responsibility
│  ├─ Perform operation on object structure? → Visitor
│  └─ Share complex state/operations? → Flyweight
│
└─ Not sure? → See "I Need To..." scenarios below
```

## "I Need To..." Scenarios

### Object Creation

| I Need To... | Use Pattern | Why |
|-------------|-------------|-----|
| Create objects without knowing exact type | Factory Method | Defers instantiation to subclasses |
| Build complex object step-by-step | Builder | Separates construction from representation |
| Ensure only one instance exists | Singleton | Controls instance creation |
| Create families of related objects | Abstract Factory | Ensures compatible object sets |
| Clone expensive objects | Prototype | Avoids repeated initialization |
| Hide complex initialization | Factory/Builder | Encapsulates complexity |

### Structural Changes

| I Need To... | Use Pattern | Why |
|-------------|-------------|-----|
| Add features without changing code | Decorator | Wraps objects with new behavior |
| Make incompatible interfaces work | Adapter | Converts interface to expected one |
| Simplify complex system | Facade | Provides unified interface |
| Reduce memory by sharing data | Flyweight | Shares common state |
| Separate interface from implementation | Bridge | Allows independent variation |
| Build tree-like structures | Composite | Treats objects uniformly |
| Lazy-load expensive objects | Proxy | Controls access, delays creation |

### Behavioral Changes

| I Need To... | Use Pattern | Why |
|-------------|-------------|-----|
| Notify many objects of changes | Observer | Decouples publishers from subscribers |
| Switch algorithms at runtime | Strategy | Encapsulates algorithms |
| Queue, log, or undo operations | Command | Encapsulates operations as objects |
| Loop through collection | Iterator | Decouples traversal from structure |
| Save and restore state | Memento | Captures state without exposing internals |
| Change behavior by state | State | Organizes state-specific behavior |
| Reduce coupling between objects | Mediator | Centralizes communication |
| Define algorithm structure | Template Method | Lets subclasses override steps |
| Handle request by multiple objects | Chain of Responsibility | Passes request along chain |
| Perform operations on object structure | Visitor | Separates operations from objects |

## Problem → Pattern Mapping

### Configuration & Settings

| Problem | Pattern(s) | Example |
|---------|-----------|---------|
| Single config object | Singleton | Database config, app settings |
| Different configs per environment | Factory Method | Dev/staging/prod configs |
| Complex config building | Builder | Multi-step config with validation |

### Data Access & APIs

| Problem | Pattern(s) | Example |
|---------|-----------|---------|
| Database connection | Singleton + Factory | Connection pool manager |
| API client | Facade + Adapter | Simplify complex API |
| Multiple data sources | Strategy + Factory | Switch between SQL/NoSQL/API |
| Result set pagination | Iterator | Cursor-based iteration |
| Query building | Builder | Fluent query interface |

### UI & Frontend

| Problem | Pattern(s) | Example |
|---------|-----------|---------|
| UI component variants | Decorator | Add tooltips, badges, animations |
| Theme switching | Strategy | Light/dark themes |
| Form state management | State | Draft/submitting/success/error |
| Undo/redo | Command + Memento | Text editor, drawing app |
| Event handling | Observer | Event listeners, state updates |

### Business Logic

| Problem | Pattern(s) | Example |
|---------|-----------|---------|
| Validation rules | Strategy + Chain of Responsibility | Sequential validators |
| Workflow steps | State | Order processing states |
| Notification system | Observer | Email/SMS/push notifications |
| Payment processing | Strategy + Command | Multiple payment methods |
| Approval workflow | Chain of Responsibility | Multi-level approvals |

### System Integration

| Problem | Pattern(s) | Example |
|---------|-----------|---------|
| Legacy system integration | Adapter + Facade | Wrap old API |
| Multiple APIs | Facade | Unified interface |
| Service communication | Mediator + Observer | Microservices coordination |
| Cache management | Proxy | Lazy load + caching |

## Pattern Combinations That Work Well

### The Cache Stack
```
Proxy (controls access)
  → Flyweight (shares data)
    → Singleton (single cache instance)
```
**Use for:** In-memory caching, resource pooling

### The Flexible Factory
```
Abstract Factory (creates families)
  → Factory Method (creates instances)
    → Builder (constructs complex objects)
```
**Use for:** Multi-tenant systems, plugin architectures

### The Event Pipeline
```
Observer (notifies listeners)
  → Chain of Responsibility (processes event)
    → Command (encapsulates action)
      → Memento (saves state)
```
**Use for:** Event-driven systems, audit trails

### The Smart Collection
```
Composite (tree structure)
  → Iterator (traversal)
    → Visitor (operations on nodes)
```
**Use for:** File systems, DOM manipulation, AST processing

### The Flexible UI
```
Composite (UI hierarchy)
  → Decorator (add features)
    → Strategy (swap behaviors)
      → Observer (update on changes)
```
**Use for:** Component libraries, admin panels

### The Service Layer
```
Facade (simple interface)
  → Adapter (normalize APIs)
    → Proxy (control access)
      → Singleton (shared instance)
```
**Use for:** API gateways, service wrappers

### The State Machine
```
State (manage states)
  → Command (encapsulate transitions)
    → Memento (save state history)
      → Observer (notify state changes)
```
**Use for:** Workflow engines, game logic

## Anti-Patterns to Avoid

### Over-Engineering
- Don't use patterns just because they exist
- Start simple, refactor to patterns when needed
- YAGNI (You Aren't Gonna Need It) principle applies

### Pattern Abuse

| Anti-Pattern | Problem | Better Solution |
|-------------|---------|-----------------|
| Singleton Everything | Global state, tight coupling | Dependency injection |
| Deep Decorator Chains | Hard to debug, performance hit | Composition or Strategy |
| Observer Overload | Memory leaks, cascade updates | Direct calls or Mediator |
| God Factory | One factory does everything | Multiple specialized factories |
| Strategy for Everything | Too many small classes | Simple if/switch first |

## When NOT to Use Patterns

| Situation | Why Skip Patterns | What To Do Instead |
|-----------|------------------|-------------------|
| Simple CRUD | Patterns add overhead | Direct implementation |
| Prototype/MVP | Need to move fast | Clean code, refactor later |
| One-time script | Won't be maintained | Straightforward procedural code |
| Team unfamiliar with patterns | Learning curve too steep | Start with simpler abstractions |
| Performance critical | Pattern indirection adds cost | Optimized direct code |

## Pattern Selection Checklist

Before choosing a pattern, ask:

1. **Complexity**: Does this problem warrant a pattern?
   - [ ] Yes: Problem is complex or likely to grow
   - [ ] No: Simple solution works fine

2. **Change**: What is likely to change?
   - [ ] How objects are created → Creational patterns
   - [ ] How objects are composed → Structural patterns
   - [ ] How objects interact → Behavioral patterns

3. **Team**: Will the team understand it?
   - [ ] Yes: Team is familiar or can learn
   - [ ] No: Consider simpler approach

4. **Maintenance**: Will this make maintenance easier?
   - [ ] Yes: Reduces coupling, improves testability
   - [ ] No: Reconsider if adding complexity

5. **Performance**: Can we afford the overhead?
   - [ ] Yes: Overhead is negligible
   - [ ] No: Profile and optimize

## Real-World Examples

### E-commerce Application

```
Product Creation:
- Factory Method → Create different product types
- Builder → Build complex product configurations
- Prototype → Clone product templates

Shopping Cart:
- Composite → Cart items (simple + bundles)
- Decorator → Add gift wrap, expedited shipping
- Strategy → Different pricing strategies

Checkout:
- State → Order states (cart → payment → processing → shipped)
- Command → Payment commands (process, refund, cancel)
- Observer → Notify customer, inventory, shipping

Payment:
- Strategy → Different payment methods
- Adapter → Third-party payment gateways
- Facade → Simplify payment process
```

### Content Management System

```
Content Types:
- Factory Method → Create different content types
- Composite → Pages with nested sections
- Decorator → Add SEO, caching, analytics

Rendering:
- Template Method → Define render pipeline
- Strategy → Different rendering engines
- Visitor → Transform content (markdown, sanitize)

Permissions:
- Chain of Responsibility → Check access levels
- Proxy → Control access to content
- Observer → Log access attempts
```

### Logging System

```
Logger Creation:
- Singleton → Single logger instance per category
- Factory → Create loggers for different contexts

Log Processing:
- Chain of Responsibility → Filter → Format → Append
- Strategy → Different output formats
- Observer → Multiple log destinations

Performance:
- Proxy → Lazy initialization
- Flyweight → Share format templates
```

## Quick Reference

### Most Commonly Used Patterns

1. **Factory Method** - Almost every codebase needs this
2. **Strategy** - Great for swappable algorithms
3. **Observer** - Essential for event-driven systems
4. **Decorator** - Better than inheritance for adding features
5. **Singleton** - Use sparingly, often indicates design smell

### Patterns to Learn First

1. **Factory Method** - Fundamental creation pattern
2. **Strategy** - Teaches encapsulation
3. **Observer** - Core of event systems
4. **Adapter** - Practical integration pattern
5. **Decorator** - Flexible alternative to inheritance

### Patterns to Use Carefully

1. **Singleton** - Can create global state issues
2. **Visitor** - Complex, breaks encapsulation
3. **Memento** - Memory intensive
4. **Flyweight** - Optimization, may be premature
5. **Mediator** - Can become god object

## Further Reading

- **Gang of Four Book**: Original patterns reference
- **Head First Design Patterns**: Beginner-friendly
- **Refactoring to Patterns**: When and how to apply
- **Pattern Languages of Program Design**: Advanced topics

---

**Remember**: Patterns are tools, not rules. Use them when they make your code clearer, more maintainable, and more flexible. Don't force them where they don't fit.
