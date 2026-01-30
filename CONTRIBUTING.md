# Contributing to Software Patterns

Thank you for your interest in contributing! This project aims to be a comprehensive, high-quality reference for design patterns and data structures.

## Ways to Contribute

### 1. Improve Existing Documentation

- Fix typos, grammar, or clarity issues
- Add missing edge cases or considerations
- Improve pseudocode examples
- Add more "Known Uses" from real frameworks

### 2. Add New Content

**Additional Patterns:**
- SOLID principles documentation
- GRASP patterns
- Enterprise integration patterns
- Concurrency patterns
- Functional programming patterns

**Additional Data Structures:**
- Persistent/immutable data structures
- Probabilistic data structures (Count-Min Sketch, HyperLogLog)
- Cache-oblivious data structures
- Concurrent data structures

**Language-Specific Examples:**
- PHP implementations
- TypeScript implementations
- Python implementations
- Go implementations

### 3. Create Decision Guides

- Domain-specific pattern selection guides
- Performance comparison benchmarks
- Migration guides between patterns

## Documentation Standards

### File Structure

Each pattern file should include:

```markdown
# Pattern Name

## Intent
One paragraph explaining the core purpose.

## Also Known As
Alternative names.

## Motivation
Real-world scenario (2-3 paragraphs).

## Applicability
Bullet list of when to use.

## Structure
ASCII diagram showing relationships.

## Participants
- **Name**: Role description

## Collaborations
How participants work together.

## Consequences
### Benefits
### Liabilities

## Implementation
Detailed pseudocode.

## Example
Complete working example.

## Known Uses
Real-world framework examples.

## Related Patterns
Connections to other patterns.

## When NOT to Use
Anti-patterns and warnings.
```

### Pseudocode Guidelines

- Use clear, readable syntax
- Include type hints where helpful
- Avoid language-specific idioms
- Comment complex logic
- Use realistic names (not Foo/Bar)

Example:
```pseudocode
class PaymentProcessor {
    function process(payment: Payment): Result {
        // Validate the payment first
        if not payment.isValid() {
            return Result.failure("Invalid payment")
        }

        // Process through gateway
        return gateway.charge(payment.amount)
    }
}
```

### ASCII Diagrams

Use box-drawing characters for clarity:

```
┌─────────────────┐     ┌─────────────────┐
│   Interface     │◀────│  Concrete       │
└─────────────────┘     └─────────────────┘
         ▲
         │
┌─────────────────┐
│   Client        │
└─────────────────┘
```

## Submission Process

1. **Fork** the repository
2. **Create a branch** for your changes
3. **Make your changes** following the standards above
4. **Test** that markdown renders correctly
5. **Submit a PR** with a clear description

### PR Description Template

```markdown
## What

Brief description of changes.

## Why

Motivation for the changes.

## Checklist

- [ ] Follows documentation standards
- [ ] Pseudocode is language-agnostic
- [ ] ASCII diagrams render correctly
- [ ] No spelling/grammar errors
- [ ] Links work correctly
```

## Code of Conduct

- Be respectful and constructive
- Focus on improving the content
- Welcome newcomers
- Give credit where due

## Questions?

Open an issue for:
- Clarification on standards
- Discussion of new content ideas
- Reporting problems

Thank you for helping make this resource better!
