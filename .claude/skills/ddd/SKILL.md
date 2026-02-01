# Domain-Driven Design Skill

Reference for domain modeling concepts from Eric Evans' "Domain-Driven Design: Tackling Complexity in the Heart of Software."

## Activation Triggers

Use this skill when discussing:
- Domain modeling and ubiquitous language
- Bounded contexts and context mapping
- Entities vs value objects
- Aggregates and aggregate roots
- Repository and factory patterns
- Domain events and event storming
- Anti-corruption layers

## Quick Reference

### Strategic DDD Patterns

| Pattern | Purpose | When to Use |
|---------|---------|-------------|
| Bounded Context | Define clear boundaries | Different teams, different models |
| Ubiquitous Language | Shared vocabulary | Always within a context |
| Context Map | Show relationships | Multiple bounded contexts |
| Anti-Corruption Layer | Protect from external models | Integrating legacy/external systems |

### Tactical DDD Patterns

| Pattern | Characteristics | Identity |
|---------|-----------------|----------|
| Entity | Mutable, lifecycle, identity matters | Has unique ID |
| Value Object | Immutable, no identity, equality by attributes | No ID needed |
| Aggregate | Consistency boundary, transactional unit | Has aggregate root |
| Domain Event | Something that happened, immutable | Named in past tense |

### Building Blocks

| Block | Responsibility |
|-------|----------------|
| Entity | Model with identity and lifecycle |
| Value Object | Describe characteristics |
| Aggregate | Enforce invariants, transactional boundary |
| Repository | Persist and retrieve aggregates |
| Factory | Complex object creation |
| Domain Service | Stateless domain operations |
| Application Service | Orchestrate use cases |
| Domain Event | Capture something that happened |

### Context Mapping Relationships

| Relationship | Description |
|--------------|-------------|
| Partnership | Two teams cooperate |
| Shared Kernel | Shared subset of model |
| Customer-Supplier | Upstream/downstream dependency |
| Conformist | Downstream conforms to upstream |
| Anti-Corruption Layer | Translate between models |
| Open Host Service | Published API for many consumers |
| Published Language | Well-documented interchange format |
| Separate Ways | No integration |

### Aggregate Design Rules

1. **Protect invariants** - Aggregate root enforces all rules
2. **Design for consistency** - One aggregate per transaction
3. **Reference by identity** - Don't hold direct references to other aggregates
4. **Use eventual consistency** - Between aggregates, not within
5. **Keep aggregates small** - Focused on true invariants

### Entity vs Value Object Decision

```
Does identity matter beyond its attributes?
├── YES → Entity (e.g., User, Order, Product)
└── NO → Value Object (e.g., Address, Money, DateRange)

Will instances be shared?
├── YES → Entity (needs identity for references)
└── NO → Value Object (can be replaced/copied)
```

## Directory Structure

```
ddd/
├── SKILL.md
├── strategic/
│   ├── ubiquitous-language.md
│   ├── bounded-contexts.md
│   ├── context-mapping.md
│   └── anti-corruption-layer.md
├── tactical/
│   ├── entities.md
│   ├── value-objects.md
│   ├── aggregates.md
│   ├── domain-services.md
│   └── domain-events.md
├── patterns/
│   ├── repositories.md
│   ├── factories.md
│   └── specifications.md
└── practices/
    ├── event-storming.md
    └── model-exploration.md
```

## Usage Examples

### Modeling a Domain

```
Question: "How should I model orders?"

Consider:
1. Define Ubiquitous Language - See strategic/ubiquitous-language.md
2. Identify Entities vs Value Objects - See tactical/
3. Find Aggregates - See tactical/aggregates.md
4. Design Repositories - See patterns/repositories.md
```

### Integrating Systems

```
Question: "How do I integrate with legacy system?"

Consider:
- Define Bounded Context boundary
- Use Anti-Corruption Layer
- Map contexts with Context Map
```

---

*Based on concepts from "Domain-Driven Design" by Eric Evans.*
