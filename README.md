# Software Patterns

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Claude Code Skill](https://img.shields.io/badge/Claude%20Code-Skill-blueviolet)](https://claude.ai/code)
[![Contributions Welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg)](CONTRIBUTING.md)

A comprehensive reference for **Gang of Four design patterns** and **fundamental data structures** for [Claude Code](https://claude.ai/code). Language-agnostic pseudocode examples that translate to any programming language.

## What's Included

| Category | Count | Description |
|----------|-------|-------------|
| **GoF Creational Patterns** | 5 | Abstract Factory, Builder, Factory Method, Prototype, Singleton |
| **GoF Structural Patterns** | 7 | Adapter, Bridge, Composite, Decorator, Facade, Flyweight, Proxy |
| **GoF Behavioral Patterns** | 11 | Chain of Responsibility, Command, Interpreter, Iterator, Mediator, Memento, Observer, State, Strategy, Template Method, Visitor |
| **Data Structures** | 23 | Arrays, Lists, Trees, Graphs, Hash Tables, and more |
| **Decision Guides** | 3 | Pattern selection, data structure selection, complexity cheat sheet |

**Total: 50 comprehensive documentation files (~35,000 lines)**

## Installation

### As a Claude Code Skill (Recommended)

```bash
npx skills add git@github.com:grndlvl/software-patterns.git -g
```

This installs the skill globally so it's available in all your projects.

### Project-Level Installation

```bash
npx skills add git@github.com:grndlvl/software-patterns.git
```

### Manual Installation

```bash
git clone git@github.com:grndlvl/software-patterns.git
cp -r software-patterns/.claude/skills/software-patterns ~/.claude/skills/
```

## Usage

Once installed, the skill **auto-activates** when you:
- Ask about or need to implement a design pattern
- Need to choose between patterns or data structures
- Are refactoring code and considering structural improvements
- Discuss architecture, decoupling, or extensibility
- Need to understand time/space complexity trade-offs

### Example Prompts

```
"Which design pattern should I use for creating objects without specifying their concrete classes?"

"What's the best data structure for fast lookups with frequent insertions?"

"Show me how to implement the Observer pattern"

"Compare the trade-offs between using a hash table vs a balanced tree"
```

## Contents

### GoF Design Patterns (23)

#### Creational Patterns
| Pattern | Intent |
|---------|--------|
| [Abstract Factory](.claude/skills/software-patterns/gof-creational/abstract-factory.md) | Create families of related objects |
| [Builder](.claude/skills/software-patterns/gof-creational/builder.md) | Construct complex objects step-by-step |
| [Factory Method](.claude/skills/software-patterns/gof-creational/factory-method.md) | Let subclasses decide which class to instantiate |
| [Prototype](.claude/skills/software-patterns/gof-creational/prototype.md) | Clone existing objects |
| [Singleton](.claude/skills/software-patterns/gof-creational/singleton.md) | Ensure single instance |

#### Structural Patterns
| Pattern | Intent |
|---------|--------|
| [Adapter](.claude/skills/software-patterns/gof-structural/adapter.md) | Convert interface to expected interface |
| [Bridge](.claude/skills/software-patterns/gof-structural/bridge.md) | Separate abstraction from implementation |
| [Composite](.claude/skills/software-patterns/gof-structural/composite.md) | Treat individual and groups uniformly |
| [Decorator](.claude/skills/software-patterns/gof-structural/decorator.md) | Add responsibilities dynamically |
| [Facade](.claude/skills/software-patterns/gof-structural/facade.md) | Simplified interface to subsystem |
| [Flyweight](.claude/skills/software-patterns/gof-structural/flyweight.md) | Share common state efficiently |
| [Proxy](.claude/skills/software-patterns/gof-structural/proxy.md) | Control access to object |

#### Behavioral Patterns
| Pattern | Intent |
|---------|--------|
| [Chain of Responsibility](.claude/skills/software-patterns/gof-behavioral/chain-of-responsibility.md) | Pass request along handler chain |
| [Command](.claude/skills/software-patterns/gof-behavioral/command.md) | Encapsulate request as object |
| [Interpreter](.claude/skills/software-patterns/gof-behavioral/interpreter.md) | Define grammar and interpret sentences |
| [Iterator](.claude/skills/software-patterns/gof-behavioral/iterator.md) | Sequential access without exposing internals |
| [Mediator](.claude/skills/software-patterns/gof-behavioral/mediator.md) | Centralize complex communications |
| [Memento](.claude/skills/software-patterns/gof-behavioral/memento.md) | Capture and restore object state |
| [Observer](.claude/skills/software-patterns/gof-behavioral/observer.md) | Notify dependents of state changes |
| [State](.claude/skills/software-patterns/gof-behavioral/state.md) | Alter behavior when state changes |
| [Strategy](.claude/skills/software-patterns/gof-behavioral/strategy.md) | Encapsulate interchangeable algorithms |
| [Template Method](.claude/skills/software-patterns/gof-behavioral/template-method.md) | Define skeleton, let subclasses fill in |
| [Visitor](.claude/skills/software-patterns/gof-behavioral/visitor.md) | Add operations without changing classes |

### Data Structures (23)

#### Linear
[Array](.claude/skills/software-patterns/data-structures/linear/array.md) ·
[Dynamic Array](.claude/skills/software-patterns/data-structures/linear/dynamic-array.md) ·
[Linked List](.claude/skills/software-patterns/data-structures/linear/linked-list.md) ·
[Stack](.claude/skills/software-patterns/data-structures/linear/stack.md) ·
[Queue](.claude/skills/software-patterns/data-structures/linear/queue.md) ·
[Deque](.claude/skills/software-patterns/data-structures/linear/deque.md)

#### Trees
[Binary Tree](.claude/skills/software-patterns/data-structures/trees/binary-tree.md) ·
[BST](.claude/skills/software-patterns/data-structures/trees/bst.md) ·
[AVL Tree](.claude/skills/software-patterns/data-structures/trees/avl-tree.md) ·
[Red-Black Tree](.claude/skills/software-patterns/data-structures/trees/red-black-tree.md) ·
[B-Tree](.claude/skills/software-patterns/data-structures/trees/b-tree.md) ·
[Trie](.claude/skills/software-patterns/data-structures/trees/trie.md) ·
[Heap](.claude/skills/software-patterns/data-structures/trees/heap.md)

#### Hash-Based
[Hash Table](.claude/skills/software-patterns/data-structures/hash-based/hash-table.md) ·
[Hash Set](.claude/skills/software-patterns/data-structures/hash-based/hash-set.md) ·
[Bloom Filter](.claude/skills/software-patterns/data-structures/hash-based/bloom-filter.md)

#### Graphs
[Adjacency List](.claude/skills/software-patterns/data-structures/graphs/adjacency-list.md) ·
[Adjacency Matrix](.claude/skills/software-patterns/data-structures/graphs/adjacency-matrix.md) ·
[Graph Algorithms](.claude/skills/software-patterns/data-structures/graphs/graph-algorithms.md)

#### Advanced
[Skip List](.claude/skills/software-patterns/data-structures/advanced/skip-list.md) ·
[Disjoint Set](.claude/skills/software-patterns/data-structures/advanced/disjoint-set.md) ·
[Segment Tree](.claude/skills/software-patterns/data-structures/advanced/segment-tree.md) ·
[Fenwick Tree](.claude/skills/software-patterns/data-structures/advanced/fenwick-tree.md)

### Decision Guides

- [Pattern Selection Guide](.claude/skills/software-patterns/decision-guides/pattern-selection.md) - Flowcharts and decision trees for choosing patterns
- [Data Structure Selection Guide](.claude/skills/software-patterns/decision-guides/data-structure-selection.md) - Choose the right structure by use case
- [Complexity Cheat Sheet](.claude/skills/software-patterns/decision-guides/complexity-cheat-sheet.md) - Big-O quick reference

## Documentation Format

Each pattern/structure includes:

- **Intent** - Core purpose in one paragraph
- **Motivation** - Real-world scenario explaining why it's needed
- **Structure** - ASCII diagrams showing relationships
- **Participants** - Roles and responsibilities
- **Implementation** - Language-agnostic pseudocode
- **Example** - Complete working example
- **When to Use / When NOT to Use** - Practical guidance
- **Related Patterns** - Connections to other patterns

## Language Support

All examples use language-agnostic pseudocode designed to translate easily to:

- **PHP** - Direct class mapping
- **JavaScript/TypeScript** - ES6 classes or functional patterns
- **Python** - Classes with type hints
- **Java/C#** - Direct mapping with generics
- **Go/Rust** - Adapt to language idioms

## Contributing

Contributions are welcome! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

Ideas for contributions:
- Additional patterns (SOLID, GRASP, enterprise patterns)
- Language-specific examples
- More data structures (persistent, probabilistic)
- Improvements to existing documentation
- Translations

## License

MIT License - see [LICENSE](LICENSE) for details.

## Acknowledgments

- Gang of Four: Erich Gamma, Richard Helm, Ralph Johnson, John Vlissides
- *Design Patterns: Elements of Reusable Object-Oriented Software* (1994)
- The software engineering community

---

**Made with Claude Code**
