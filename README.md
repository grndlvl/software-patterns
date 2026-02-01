# Software Patterns

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Claude Code Skill](https://img.shields.io/badge/Claude%20Code-Skill-blueviolet)](https://claude.ai/code)
[![Contributions Welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg)](CONTRIBUTING.md)

A comprehensive collection of **7 focused Claude Code skills** based on canonical software engineering resources. Each skill follows the "One Book = One Skill" principle with language-agnostic pseudocode examples.

## Skills Overview

| Skill | Source | Files | Topics |
|-------|--------|-------|--------|
| `gof-patterns` | Gang of Four | 25 | 23 design patterns + decision guides |
| `clrs-algorithms` | CLRS | 40 | Data structures & algorithms |
| `clean-code` | Robert Martin | 14 | Code quality, SOLID, refactoring |
| `ddia` | Martin Kleppmann | 21 | Distributed systems architecture |
| `pragmatic-programmer` | Hunt & Thomas | 19 | Software craftsmanship |
| `ddd` | Eric Evans | 15 | Domain modeling |
| `sicp` | Abelson & Sussman | 13 | Programming fundamentals |

**Total: 147 documentation files**

## Installation

### All Skills (Recommended)

```bash
npx skills add git@github.com:grndlvl/software-patterns.git -g
```

### Individual Skills

```bash
# Install only the skill you need
npx skills add git@github.com:grndlvl/software-patterns.git --path .claude/skills/gof-patterns -g
npx skills add git@github.com:grndlvl/software-patterns.git --path .claude/skills/clrs-algorithms -g
npx skills add git@github.com:grndlvl/software-patterns.git --path .claude/skills/clean-code -g
npx skills add git@github.com:grndlvl/software-patterns.git --path .claude/skills/ddia -g
npx skills add git@github.com:grndlvl/software-patterns.git --path .claude/skills/pragmatic-programmer -g
npx skills add git@github.com:grndlvl/software-patterns.git --path .claude/skills/ddd -g
npx skills add git@github.com:grndlvl/software-patterns.git --path .claude/skills/sicp -g
```

## Skills Detail

### GoF Patterns (`gof-patterns`)

*Based on "Design Patterns: Elements of Reusable Object-Oriented Software" by Gamma, Helm, Johnson, Vlissides*

**Creational (5):** Abstract Factory, Builder, Factory Method, Prototype, Singleton

**Structural (7):** Adapter, Bridge, Composite, Decorator, Facade, Flyweight, Proxy

**Behavioral (11):** Chain of Responsibility, Command, Interpreter, Iterator, Mediator, Memento, Observer, State, Strategy, Template Method, Visitor

**Auto-activates when:** Discussing design patterns, object creation, code structure, decoupling

---

### CLRS Algorithms (`clrs-algorithms`)

*Based on "Introduction to Algorithms" by Cormen, Leiserson, Rivest, Stein*

**Linear:** Array, Dynamic Array, Linked List, Stack, Queue, Deque

**Trees:** Binary Tree, BST, AVL, Red-Black, B-Tree, Trie, Heap, Interval Tree, Order-Statistic Tree, Splay Tree, Treap, vEB Tree

**Hash-Based:** Hash Table, Hash Set, Bloom Filter

**Graphs:** Adjacency List/Matrix, BFS, DFS, Dijkstra, Bellman-Ford, Floyd-Warshall, MST (Prim/Kruskal), Topological Sort, SCC, Network Flow

**Advanced:** Skip List, Disjoint Set, Segment Tree, Fenwick Tree, Binomial/Fibonacci Heap

**Strings:** KMP, Rabin-Karp, Suffix Array, Suffix Tree

**Auto-activates when:** Choosing data structures, analyzing complexity, implementing algorithms

---

### Clean Code (`clean-code`)

*Based on "Clean Code" by Robert C. Martin*

**SOLID Principles:** Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion

**Practices:** Meaningful names, small functions, comments, formatting, error handling, unit testing (F.I.R.S.T.), code smells, Boy Scout Rule

**Auto-activates when:** Reviewing code quality, refactoring, discussing best practices

---

### DDIA (`ddia`)

*Based on "Designing Data-Intensive Applications" by Martin Kleppmann*

**Data Models:** Relational, Document, Graph

**Storage:** B-Trees, LSM-Trees, Column Storage

**Replication:** Leader-Follower, Multi-Leader, Leaderless

**Partitioning:** Strategies, Rebalancing

**Transactions:** ACID, Isolation Levels, Distributed Transactions

**Consistency:** Models, Linearizability, Consensus Algorithms

**Processing:** Batch, Stream, Event Sourcing/CQRS

**Auto-activates when:** Designing distributed systems, choosing databases, discussing consistency/availability

---

### Pragmatic Programmer (`pragmatic-programmer`)

*Based on "The Pragmatic Programmer" by Hunt and Thomas*

**Principles:** DRY, Orthogonality, Reversibility, Tracer Bullets, Prototypes, Estimating, Domain Languages

**Practices:** Plain Text, Shell Games, Debugging, Text Manipulation, Code Generators, Design by Contract, Assertive Programming, Decoupling, Refactoring, Testing, Automation

**Auto-activates when:** Discussing software craftsmanship, pragmatic approaches, developer productivity

---

### DDD (`ddd`)

*Based on "Domain-Driven Design" by Eric Evans*

**Strategic:** Ubiquitous Language, Bounded Contexts, Context Mapping, Anti-Corruption Layer

**Tactical:** Entities, Value Objects, Aggregates, Domain Services, Domain Events

**Patterns:** Repositories, Factories, Specifications

**Practices:** Event Storming, Model Exploration

**Auto-activates when:** Modeling domains, discussing bounded contexts, designing aggregates

---

### SICP (`sicp`)

*Based on "Structure and Interpretation of Computer Programs" by Abelson and Sussman*

**Procedures:** Abstraction, Higher-Order Functions, Recursion Patterns (linear, tail, tree, mutual)

**Data:** Data Abstraction, Hierarchical Data, Symbolic Data

**Modularity:** Assignment and State, Environment Model, Streams

**Metalinguistic:** Interpreters, Lazy Evaluation, Register Machines

**Auto-activates when:** Discussing abstraction, functional programming, interpreters, compilation

---

## Documentation Format

Each topic includes:

- **Definition/Intent** - Core concept in one paragraph
- **When to Use** - Practical guidance with decision trees
- **Implementation** - Language-agnostic pseudocode
- **Examples** - Complete working examples
- **Trade-offs** - Comparison tables and alternatives
- **Summary** - Quick reference tables

## Language Support

All examples use language-agnostic pseudocode designed to translate easily to:

- **PHP** - Direct class mapping
- **JavaScript/TypeScript** - ES6 classes or functional patterns
- **Python** - Classes with type hints
- **Java/C#** - Direct mapping with generics
- **Go/Rust** - Adapt to language idioms

## Contributing

Contributions welcome! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

Ideas:
- Additional patterns or algorithms
- Language-specific example translations
- Improvements to existing documentation
- New skills from other canonical resources

## License

MIT License - see [LICENSE](LICENSE) for details.

## Acknowledgments

- **Gang of Four:** Gamma, Helm, Johnson, Vlissides
- **CLRS:** Cormen, Leiserson, Rivest, Stein
- **Robert C. Martin** (Uncle Bob)
- **Martin Kleppmann**
- **Eric Evans**
- **Andrew Hunt & David Thomas**
- **Harold Abelson & Gerald Jay Sussman**

---

**Made with Claude Code**
