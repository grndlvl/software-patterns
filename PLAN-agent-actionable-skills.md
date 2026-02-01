# Plan: Agent-Actionable Software Patterns Knowledge Base

> **Status:** Ready for implementation
> **Created:** 2025-02-01
> **Resume:** Pick up this plan to implement the agent-actionable query system

## Goal

Transform the 147-file software patterns knowledge base from a passive reference into an **active guidance system** that:
1. Agents can **query** for recommendations ("What DS for X?")
2. Agents **proactively propose** patterns/algorithms during planning
3. Engineers get **cross-skill orchestration** for complex problems
4. Implementation guidance goes from "use X" to **working code**

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    software-patterns (Router Skill)              │
│  - Receives queries and problem descriptions                     │
│  - Routes to appropriate skills based on problem taxonomy        │
│  - Orchestrates cross-skill solutions                            │
└─────────────────────────────────────────────────────────────────┘
          │                    │                    │
          ▼                    ▼                    ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│  problem-       │  │  decision-      │  │  composite-     │
│  taxonomy.yaml  │  │  trees/*.yaml   │  │  solutions.yaml │
│  (semantic      │  │  (machine-      │  │  (multi-skill   │
│   matching)     │  │   navigable)    │  │   stacks)       │
└─────────────────┘  └─────────────────┘  └─────────────────┘
          │                    │                    │
          └────────────────────┼────────────────────┘
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│  Existing Skills: gof-patterns, clrs-algorithms, ddia, ddd,     │
│                   clean-code, pragmatic-programmer, sicp        │
└─────────────────────────────────────────────────────────────────┘
```

## Design Decisions (Confirmed)

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Data format | **YAML** | More readable, easier to maintain for knowledge base content |
| Existing ASCII guides | **Keep both** | ASCII for human reading, YAML for machine navigation |
| Proactive suggestions | **Moderate** | Suggest during planning phases when clear indicators present |
| Coverage | **100%** | Every file in all 7 skills indexed - nothing orphaned |

## Coverage Requirements

**All 147 files must be indexed in problem-taxonomy.yaml:**

| Skill | Files | Items to Index |
|-------|-------|----------------|
| gof-patterns | 25 | 23 patterns (creational, structural, behavioral) |
| clrs-algorithms | 40 | ~35 data structures + sorting/graph/string algorithms |
| clean-code | 14 | 5 SOLID principles + 8 practices |
| ddia | 21 | 3 data models + 3 storage + 3 replication + partitioning + transactions + consistency + processing |
| pragmatic-programmer | 19 | 7 principles + 11 practices |
| ddd | 15 | 4 strategic + 5 tactical + 3 patterns + 2 practices |
| sicp | 13 | 3 procedures + 3 data + 3 modularity + 3 metalinguistic |
| **Total** | **147** | **~150 taxonomy entries** |

Each entry includes:
- Unique ID
- Keywords for semantic matching
- Question formulation ("Need to X?")
- Skill + file reference
- Related entries (cross-links)
- Constraints/considerations

## Implementation Phases

### Phase 1: Router Skill + Problem Taxonomy (Core)

**Create new router skill that unifies access to all 7 skills:**

| File | Purpose |
|------|---------|
| `.claude/skills/software-patterns/SKILL.md` | Main router skill with query interface |
| `.claude/skills/software-patterns/index/problem-taxonomy.yaml` | Problem-to-solution mappings |
| `.claude/skills/software-patterns/index/composite-solutions.yaml` | Multi-skill solution stacks |

**Problem Taxonomy Structure:**
```yaml
problems:
  - id: fast-key-lookup
    keywords: ["lookup", "dictionary", "key-value", "cache"]
    question: "Need fast lookup by key?"
    skills:
      - skill: clrs-algorithms
        structures: [hash-table]
        relevance: primary
    constraints:
      - "Need ordering? → tree-based instead"
```

**Composite Solutions Structure:**
```yaml
composite_solutions:
  - id: caching-system
    problem_indicators: ["cache", "memoization"]
    solution_stack:
      - skill: gof-patterns
        patterns: [proxy, flyweight]
        role: "Access control and sharing"
      - skill: clrs-algorithms
        structures: [hash-table, linked-list]
        role: "LRU eviction"
      - skill: ddia
        topics: [consistency-models]
        role: "Invalidation strategy"
```

### Phase 2: Structured Decision Trees

**Convert ASCII decision trees to machine-navigable YAML:**

| File | Source |
|------|--------|
| `index/decision-trees/data-structure-selection.yaml` | From clrs-algorithms |
| `index/decision-trees/pattern-selection.yaml` | From gof-patterns |
| `index/decision-trees/storage-engine-selection.yaml` | From ddia |

**Decision Tree Structure:**
```yaml
decision_tree:
  root:
    question: "What is your primary operation?"
    branches:
      - answer: "Fast lookup by key"
        next: key_lookup
      - answer: "Maintain sorted order"
        next: sorted_order
  key_lookup:
    question: "Do you need sorted iteration?"
    branches:
      - answer: "No"
        recommendation:
          structure: hash-table
          file: clrs-algorithms/data-structures/hash-based/hash-table.md
          complexity: "O(1) average"
```

### Phase 3: Proactive Triggers

**Enable agents to suggest patterns during planning:**

| File | Purpose |
|------|---------|
| `index/triggers.yaml` | Proactive suggestion rules |

**Trigger Structure:**
```yaml
proactive_triggers:
  - trigger: conditional_explosion
    indicators: ["many if/else", "switch with 5+ cases", "type checking"]
    suggestion:
      message: "Consider Strategy or State pattern"
      patterns: [strategy, state]

  - trigger: planning_new_feature
    context: ["planning", "designing", "architecture"]
    actions:
      - "Analyze for applicable patterns"
      - "Check problem taxonomy"
      - "Suggest cross-skill solutions"
```

### Phase 4: Implementation Guidance

**Go from "use X" to working code:**

| File | Purpose |
|------|---------|
| `templates/pattern-implementation.md` | How to generate implementations |
| `templates/language-idioms.md` | Language-specific conventions |
| `quick-ref/pattern-cheatsheet.md` | One-liner pattern guide |
| `quick-ref/ds-cheatsheet.md` | Data structure quick reference |

### Phase 5: Query Commands (Documentation)

**Document query patterns in SKILL.md:**

| Query Pattern | Example | Response |
|---------------|---------|----------|
| `/pattern <problem>` | `/pattern undo-redo` | Command + Memento patterns |
| `/ds <requirement>` | `/ds fast-lookup ordered` | Red-Black Tree or Skip List |
| `/architecture <scenario>` | `/architecture caching` | Multi-skill solution stack |
| `/implement <pattern> <lang>` | `/implement strategy typescript` | Full implementation |
| `/compare <a> vs <b>` | `/compare hash-table vs tree-map` | Trade-off analysis |

## File Structure (New)

```
.claude/skills/software-patterns/
├── SKILL.md                              # Router skill (NEW)
├── index/
│   ├── problem-taxonomy.yaml             # Problem→solution mappings (NEW)
│   ├── composite-solutions.yaml          # Multi-skill stacks (NEW)
│   ├── triggers.yaml                     # Proactive triggers (NEW)
│   └── decision-trees/
│       ├── data-structure-selection.yaml # (NEW)
│       ├── pattern-selection.yaml        # (NEW)
│       └── storage-engine-selection.yaml # (NEW)
├── templates/
│   ├── pattern-implementation.md         # (NEW)
│   └── language-idioms.md                # (NEW)
├── quick-ref/
│   ├── pattern-cheatsheet.md             # (NEW)
│   └── ds-cheatsheet.md                  # (NEW)
└── examples/
    └── composite-solutions/              # (NEW)
        ├── caching-example.md
        └── event-system-example.md
```

## How It Works

### Example 1: Direct Query
```
User: "What data structure for fast key-value lookup with ordered iteration?"

Agent:
1. Activates software-patterns skill
2. Parses: need hash-like O(1) + ordering (constraint conflict)
3. Loads decision-tree: data-structure-selection.yaml
4. Returns: "Red-Black Tree or Skip List (O(log n) but ordered)"
5. Links to: clrs-algorithms/data-structures/trees/red-black-tree.md
```

### Example 2: Proactive During Planning
```
User: "I'm planning a notification system for email, SMS, and push"

Agent:
1. Detects planning context + multiple variants
2. Proactive trigger fires: "multiple implementations of same interface"
3. Suggests: "Consider Strategy pattern for notification channels"
4. Offers implementation guidance if requested
```

### Example 3: Cross-Skill Orchestration
```
User: "How should I implement caching for my API?"

Agent:
1. Identifies "caching" as composite solution
2. Loads composite-solutions.yaml
3. Returns multi-skill guidance:
   - gof-patterns: Proxy for access control
   - clrs-algorithms: Hash + LinkedList for LRU
   - ddia: Consistency model considerations
```

## Priority & Effort

| Phase | Files | Effort | Priority |
|-------|-------|--------|----------|
| 1 | SKILL.md, problem-taxonomy.yaml (~150 entries), composite-solutions.yaml | **High** | **Critical** |
| 2 | 3 decision tree YAMLs (full coverage of decision paths) | Medium | High |
| 3 | triggers.yaml | Low | Medium |
| 4 | templates + quick-ref (4 files) | Medium | Medium |
| 5 | examples (2 files) | Low | Low |

**Total: ~14 new files, ~150 taxonomy entries for 100% coverage**

### Phase 1 Breakdown (100% Coverage)

| Skill | Entries | Complexity |
|-------|---------|------------|
| gof-patterns | 23 | Medium (well-defined patterns) |
| clrs-algorithms | 35 | High (many cross-references) |
| clean-code | 13 | Low (clear principles) |
| ddia | 18 | High (interconnected topics) |
| pragmatic-programmer | 18 | Low (distinct practices) |
| ddd | 14 | Medium (strategic/tactical split) |
| sicp | 12 | Medium (conceptual) |
| **Composite solutions** | ~15 | High (multi-skill orchestration) |
| **Total** | **~148** | |

## Verification

After implementation:

1. **Query Test**: Ask "What pattern for undo/redo?" → Should return Command + Memento
2. **DS Test**: Ask "Best structure for priority queue?" → Should return Heap
3. **Composite Test**: Ask "How to implement caching?" → Should return multi-skill stack
4. **Proactive Test**: Describe planning a "payment system with multiple providers" → Should suggest Strategy pattern
5. **Implementation Test**: Ask "Implement Strategy pattern in TypeScript" → Should return complete code

---

## To Resume This Plan

Run: `implement the plan in PLAN-agent-actionable-skills.md`
