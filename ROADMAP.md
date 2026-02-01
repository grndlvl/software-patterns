# Software Patterns Skills Roadmap

## Overview

Plan to split the current `software-patterns` skill into focused, single-source skills following the "One Book = One Skill" principle.

## Phase 1: Split Current Skill ✅ COMPLETE

- [x] **Create `gof-patterns` skill** ✅
  - Move `gof-creational/` (5 patterns)
  - Move `gof-structural/` (7 patterns)
  - Move `gof-behavioral/` (11 patterns)
  - Create SKILL.md with pattern-focused activation triggers
  - Include `decision-guides/pattern-selection.md`
  - Reference: "Design Patterns: Elements of Reusable Object-Oriented Software" (Gang of Four)
  - **Location:** `.claude/skills/gof-patterns/` (25 files)

- [x] **Create `clrs-algorithms` skill** ✅
  - Move `data-structures/` (37 files: linear, trees, hash-based, graphs, strings, advanced, algorithms)
  - Create SKILL.md with algorithm/data structure-focused activation triggers
  - Include `decision-guides/data-structure-selection.md` and `complexity-cheat-sheet.md`
  - Reference: "Introduction to Algorithms" (CLRS)
  - **Location:** `.claude/skills/clrs-algorithms/` (40 files)

- [x] **Remove original `software-patterns` skill** ✅
  - Both new skills created and verified
  - Original directory removed

## Phase 2: New Biblical Resources

- [x] **Create `clean-code` skill** ✅
  - SOLID Principles (SRP, OCP, LSP, ISP, DIP)
  - Meaningful names (classes, methods, variables)
  - Functions (small, single purpose, command-query separation)
  - Comments (when to use, when to avoid)
  - Formatting and code organization
  - Error handling patterns
  - Unit testing principles (F.I.R.S.T.)
  - Code smells and refactoring patterns
  - Boy Scout Rule
  - Reference: "Clean Code" by Robert Martin
  - **Location:** `.claude/skills/clean-code/` (14 files)

- [ ] **Create `ddia` skill**
  - Data models (relational, document, graph)
  - Storage engines (LSM trees, B-trees)
  - Replication strategies (leader-follower, multi-leader, leaderless)
  - Partitioning/sharding patterns
  - Transactions and consistency models (ACID, BASE, linearizability)
  - Consensus algorithms (Paxos, Raft)
  - Stream processing vs batch processing
  - Event sourcing and CQRS
  - Reference: "Designing Data-Intensive Applications" by Martin Kleppmann

- [ ] **Create `ddd` skill**
  - Ubiquitous Language
  - Bounded Contexts and Context Maps
  - Entities vs Value Objects
  - Aggregates and Aggregate Roots
  - Repositories pattern
  - Domain Services vs Application Services
  - Domain Events
  - Factories pattern
  - Anti-corruption Layer
  - Strategic vs Tactical DDD
  - Event Storming basics
  - Reference: "Domain-Driven Design" by Eric Evans

- [x] **Create `pragmatic-programmer` skill** ✅
  - DRY (Don't Repeat Yourself) principle
  - Orthogonality and decoupling
  - Tracer bullets and prototyping
  - Domain languages
  - Estimating and planning
  - Debugging strategies
  - Code generators
  - Design by Contract
  - Assertive programming
  - When to refactor
  - Testing strategies
  - Automation principles
  - Reference: "The Pragmatic Programmer" by Hunt and Thomas
  - **Location:** `.claude/skills/pragmatic-programmer/` (19 files)

- [ ] **Create `sicp` skill**
  - Abstraction with procedures
  - Higher-order functions
  - Data abstraction and representation
  - Modularity and state management
  - Streams and lazy evaluation
  - Metalinguistic abstraction (interpreters)
  - Register machines and compilation
  - Recursion patterns (tree, tail, mutual)
  - Reference: "Structure and Interpretation of Computer Programs" by Abelson and Sussman

## Final State

7 focused skills, each representing a canonical software engineering resource:

| Skill | Source Book | Focus |
|-------|-------------|-------|
| `gof-patterns` | Gang of Four | Object-oriented design patterns (23 patterns) |
| `clrs-algorithms` | CLRS | Data structures & algorithms (37+ structures) |
| `clean-code` | Robert Martin | Code quality & style |
| `ddia` | Kleppmann | Distributed systems architecture |
| `ddd` | Eric Evans | Domain modeling |
| `pragmatic-programmer` | Hunt & Thomas | Software craftsmanship |
| `sicp` | Abelson & Sussman | Programming fundamentals |
