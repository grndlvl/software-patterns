# Structure and Interpretation of Computer Programs Skill

Reference for fundamental programming concepts from Abelson and Sussman's "Structure and Interpretation of Computer Programs."

## Activation Triggers

Use this skill when discussing:
- Procedural abstraction and higher-order functions
- Data abstraction and representation
- Recursion patterns (tree, tail, mutual)
- State, assignment, and environment model
- Streams and lazy evaluation
- Interpreters and metalinguistic abstraction
- Compilation and register machines

## Quick Reference

### Core Abstractions

| Abstraction | Purpose | Key Concept |
|-------------|---------|-------------|
| Procedural | Hide implementation details | Black-box abstraction |
| Data | Separate use from representation | Constructors/selectors |
| Syntactic | Create new languages | Interpreters/macros |

### Higher-Order Function Patterns

| Pattern | Description | Example |
|---------|-------------|---------|
| Map | Transform each element | `(map square list)` |
| Filter | Select matching elements | `(filter even? list)` |
| Fold/Reduce | Accumulate to single value | `(fold + 0 list)` |
| Compose | Combine functions | `(compose f g)` |

### Recursion Patterns

| Pattern | Characteristics | Space |
|---------|-----------------|-------|
| Linear | Single recursive call | O(n) |
| Tail | Result in recursive call | O(1) |
| Tree | Multiple recursive calls | O(depth) |
| Mutual | Functions call each other | Varies |

### Data Structures

| Structure | Representation | Operations |
|-----------|----------------|------------|
| Pair | cons, car, cdr | Basic building block |
| List | Chain of pairs | Sequence operations |
| Tree | Nested pairs | Hierarchical data |
| Set | List or tree | Union, intersection |

### Evaluation Models

| Model | State | Binding |
|-------|-------|---------|
| Substitution | Stateless | Direct replacement |
| Environment | Stateful | Frame-based lookup |

### Stream Operations

| Operation | Behavior | Evaluation |
|-----------|----------|------------|
| cons-stream | Delay tail | Lazy |
| stream-car | Get first | Immediate |
| stream-cdr | Force tail | On-demand |
| stream-map | Transform lazily | Lazy |
| stream-filter | Select lazily | Lazy |

## Directory Structure

```
sicp/
├── SKILL.md
├── procedures/
│   ├── abstraction.md
│   ├── higher-order-functions.md
│   └── recursion-patterns.md
├── data/
│   ├── data-abstraction.md
│   ├── hierarchical-data.md
│   └── symbolic-data.md
├── modularity/
│   ├── assignment-and-state.md
│   ├── environment-model.md
│   └── streams.md
└── metalinguistic/
    ├── interpreters.md
    ├── lazy-evaluation.md
    └── register-machines.md
```

## Usage Examples

### Designing Abstractions

```
Question: "How should I structure this complex function?"

Consider:
1. Identify primitive operations
2. Build compound procedures - See procedures/abstraction.md
3. Use higher-order functions - See procedures/higher-order-functions.md
4. Choose appropriate recursion - See procedures/recursion-patterns.md
```

### Managing Complexity

```
Question: "How do I handle growing complexity?"

Consider:
- Data abstraction barriers - See data/data-abstraction.md
- Modularity through state - See modularity/assignment-and-state.md
- Lazy evaluation for infinite data - See modularity/streams.md
```

### Building Languages

```
Question: "Should I create a DSL?"

Consider:
- Interpreter design - See metalinguistic/interpreters.md
- Lazy evaluation semantics - See metalinguistic/lazy-evaluation.md
- Compilation strategies - See metalinguistic/register-machines.md
```

---

*Based on concepts from "Structure and Interpretation of Computer Programs" by Harold Abelson and Gerald Jay Sussman.*
