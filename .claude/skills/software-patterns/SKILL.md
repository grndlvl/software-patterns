---
name: software-patterns
description: "Gang of Four design patterns and data structures reference. Use this skill when implementing, discussing, or choosing design patterns or data structures. Auto-activates for architecture decisions, refactoring, and algorithm selection. Comprehensive coverage of all 23 GoF patterns and major data structures with pseudocode examples."
---

# Software Patterns - Design Patterns & Data Structures Reference

A comprehensive reference for the Gang of Four design patterns and fundamental data structures. This skill provides language-agnostic guidance with pseudocode examples that can be translated to any programming language.

## When This Skill Activates

This skill automatically activates when you:
- Ask about or need to implement a design pattern
- Need to choose between patterns or data structures
- Are refactoring code and considering structural improvements
- Discuss architecture, decoupling, or extensibility
- Need to understand time/space complexity trade-offs

## Quick Pattern Reference

### Creational Patterns (Object Creation)
| Pattern | Intent | Use When |
|---------|--------|----------|
| [Abstract Factory](gof-creational/abstract-factory.md) | Create families of related objects | Need platform/theme independence |
| [Builder](gof-creational/builder.md) | Construct complex objects step-by-step | Object has many optional parts |
| [Factory Method](gof-creational/factory-method.md) | Let subclasses decide which class to instantiate | Don't know concrete types ahead of time |
| [Prototype](gof-creational/prototype.md) | Clone existing objects | Object creation is expensive |
| [Singleton](gof-creational/singleton.md) | Ensure single instance | Need exactly one shared instance |

### Structural Patterns (Composition)
| Pattern | Intent | Use When |
|---------|--------|----------|
| [Adapter](gof-structural/adapter.md) | Convert interface to expected interface | Integrating incompatible interfaces |
| [Bridge](gof-structural/bridge.md) | Separate abstraction from implementation | Need to vary both independently |
| [Composite](gof-structural/composite.md) | Treat individual and groups uniformly | Have tree structures |
| [Decorator](gof-structural/decorator.md) | Add responsibilities dynamically | Need flexible extension |
| [Facade](gof-structural/facade.md) | Simplified interface to subsystem | Complex subsystem needs simple API |
| [Flyweight](gof-structural/flyweight.md) | Share common state efficiently | Many similar objects needed |
| [Proxy](gof-structural/proxy.md) | Control access to object | Need lazy loading, access control, logging |

### Behavioral Patterns (Communication)
| Pattern | Intent | Use When |
|---------|--------|----------|
| [Chain of Responsibility](gof-behavioral/chain-of-responsibility.md) | Pass request along handler chain | Multiple handlers, unknown which handles |
| [Command](gof-behavioral/command.md) | Encapsulate request as object | Need undo, queue, or log operations |
| [Interpreter](gof-behavioral/interpreter.md) | Define grammar and interpret sentences | Have a simple language to parse |
| [Iterator](gof-behavioral/iterator.md) | Sequential access without exposing internals | Need to traverse collections |
| [Mediator](gof-behavioral/mediator.md) | Centralize complex communications | Many objects communicate in complex ways |
| [Memento](gof-behavioral/memento.md) | Capture and restore object state | Need undo/snapshot functionality |
| [Observer](gof-behavioral/observer.md) | Notify dependents of state changes | One-to-many dependency |
| [State](gof-behavioral/state.md) | Alter behavior when state changes | Object behavior depends on state |
| [Strategy](gof-behavioral/strategy.md) | Encapsulate interchangeable algorithms | Need to swap algorithms at runtime |
| [Template Method](gof-behavioral/template-method.md) | Define skeleton, let subclasses fill in | Algorithm structure fixed, steps vary |
| [Visitor](gof-behavioral/visitor.md) | Add operations without changing classes | Need to add many operations to stable structure |

## Quick Data Structure Reference

### Linear Structures
| Structure | Access | Search | Insert | Delete | Use When |
|-----------|--------|--------|--------|--------|----------|
| [Array](data-structures/linear/array.md) | O(1) | O(n) | O(n) | O(n) | Known size, index access |
| [Dynamic Array](data-structures/linear/dynamic-array.md) | O(1) | O(n) | O(1)* | O(n) | Unknown size, frequent append |
| [Linked List](data-structures/linear/linked-list.md) | O(n) | O(n) | O(1) | O(1) | Frequent insert/delete |
| [Stack](data-structures/linear/stack.md) | O(1) | O(n) | O(1) | O(1) | LIFO needed |
| [Queue](data-structures/linear/queue.md) | O(1) | O(n) | O(1) | O(1) | FIFO needed |
| [Deque](data-structures/linear/deque.md) | O(1) | O(n) | O(1) | O(1) | Both ends access |

### Trees
| Structure | Search | Insert | Delete | Use When |
|-----------|--------|--------|--------|----------|
| [Binary Search Tree](data-structures/trees/bst.md) | O(log n)* | O(log n)* | O(log n)* | Ordered data, frequent search |
| [AVL Tree](data-structures/trees/avl-tree.md) | O(log n) | O(log n) | O(log n) | Guaranteed balance needed |
| [Red-Black Tree](data-structures/trees/red-black-tree.md) | O(log n) | O(log n) | O(log n) | Frequent inserts/deletes |
| [B-Tree](data-structures/trees/b-tree.md) | O(log n) | O(log n) | O(log n) | Disk-based storage |
| [Trie](data-structures/trees/trie.md) | O(m) | O(m) | O(m) | String/prefix operations |
| [Heap](data-structures/trees/heap.md) | O(1)/O(n) | O(log n) | O(log n) | Priority queue needed |

### Hash-Based
| Structure | Search | Insert | Delete | Use When |
|-----------|--------|--------|--------|----------|
| [Hash Table](data-structures/hash-based/hash-table.md) | O(1)* | O(1)* | O(1)* | Fast key-value lookup |
| [Hash Set](data-structures/hash-based/hash-set.md) | O(1)* | O(1)* | O(1)* | Unique membership testing |
| [Bloom Filter](data-structures/hash-based/bloom-filter.md) | O(k) | O(k) | N/A | Probabilistic membership |

### Graphs
| Structure | Space | Add Edge | Query Edge | Use When |
|-----------|-------|----------|------------|----------|
| [Adjacency List](data-structures/graphs/adjacency-list.md) | O(V+E) | O(1) | O(degree) | Sparse graphs |
| [Adjacency Matrix](data-structures/graphs/adjacency-matrix.md) | O(V²) | O(1) | O(1) | Dense graphs |

### Advanced
| Structure | Use When |
|-----------|----------|
| [Skip List](data-structures/advanced/skip-list.md) | Probabilistic balanced list |
| [Disjoint Set](data-structures/advanced/disjoint-set.md) | Union-find operations |
| [Segment Tree](data-structures/advanced/segment-tree.md) | Range queries with updates |
| [Fenwick Tree](data-structures/advanced/fenwick-tree.md) | Prefix sums with updates |

*\* = amortized or average case*

## Decision Guides

- [Which Pattern Should I Use?](decision-guides/pattern-selection.md) - Flowchart for pattern selection
- [Which Data Structure Should I Use?](decision-guides/data-structure-selection.md) - Decision guide by use case
- [Complexity Cheat Sheet](decision-guides/complexity-cheat-sheet.md) - Quick reference for Big-O

## How to Use This Reference

1. **Choosing a pattern/structure**: Start with the decision guides
2. **Learning a pattern**: Read the full documentation with examples
3. **Quick reminder**: Use the tables above for at-a-glance reference
4. **Implementation**: Follow the pseudocode, adapt to your language

## Language Translation Notes

The pseudocode in this reference uses these conventions:
- `class` for type definitions
- `function` for methods/functions
- `->` for method calls on objects
- `//` for comments
- Type hints shown as `name: Type`

Translate to your language:
- **PHP**: `class`, `function`, `->`, `//`, type hints in docblocks or PHP 8+
- **JavaScript/TypeScript**: `class`, `function`/arrow, `.`, `//`, TS types
- **Python**: `class`, `def`, `.`, `#`, type hints
- **Java/C#**: Direct mapping with `new`, generics
