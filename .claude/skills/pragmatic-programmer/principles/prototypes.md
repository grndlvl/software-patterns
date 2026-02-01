# Prototypes and Post-It Notes

## Definition

> "Prototype to learn."
> — Dave Thomas & Andy Hunt

Prototypes are experiments designed to answer questions. They're meant to be thrown away—the value is in the learning, not the code.

## Prototypes vs. Tracer Bullets

| Aspect | Prototype | Tracer Bullet |
|--------|-----------|---------------|
| Purpose | Learn and explore | Build incrementally |
| Quality | Throwaway | Production-ready |
| Fate | Discarded | Evolves into final system |
| Scope | Answer specific question | End-to-end skeleton |
| Detail | Ignores unimportant parts | Thin but complete |

## What to Prototype

### Architecture

- How will components communicate?
- Can we achieve required performance?
- Will this scale?

### New Technology

- Can we integrate this library?
- Does this service meet our needs?
- How steep is the learning curve?

### User Interface

- Is this workflow intuitive?
- What information do users need?
- How should data be presented?

### Algorithms

- Is this approach fast enough?
- Does it handle edge cases?
- Can we reduce complexity?

### External Systems

- Can we connect to this API?
- What's the data format?
- How do we handle errors?

## How to Prototype

### 1. Identify What You're Learning

```pseudocode
// Bad: "I'll prototype the user system"
// Too vague - what's the question?

// Good: "Can our ORM handle 10,000 concurrent users?"
// Clear question to answer
```

### 2. Ignore Everything Irrelevant

```pseudocode
// Prototyping: "Can we parse this file format?"

// DON'T worry about:
// - Error handling
// - Edge cases
// - Performance
// - Clean code
// - Tests

// DO focus on:
// - Can we read the format at all?
// - Do we understand the structure?

function canWeParseThis() {
    // Quick and dirty
    file = open("sample.dat")
    data = file.read()

    // Just try to parse it
    result = magicParse(data)
    print(result)

    // Did it work? That's all we need to know.
}
```

### 3. Throw It Away

The most important step. Prototypes are:
- Not production code
- Not a head start
- Not something to "clean up later"

```pseudocode
// After prototyping:

// DELETE the prototype code
// KEEP the knowledge gained
// WRITE production code from scratch with proper architecture
```

## Prototyping Approaches

### Paper Prototypes

For UI questions, don't code at all:

```
┌──────────────────────────────────┐
│  [Logo]    Search: [________]    │
├──────────────────────────────────┤
│  Welcome, User!                  │
│                                  │
│  Recent Items:                   │
│  ┌────┐ ┌────┐ ┌────┐           │
│  │    │ │    │ │    │           │
│  └────┘ └────┘ └────┘           │
│                                  │
│  [Create New] [Browse All]       │
└──────────────────────────────────┘

User: "I'd look for the search first"
→ Learned: Search should be more prominent
```

### Spike Solutions

Time-boxed experiments:

```pseudocode
// Spike: Can we integrate with Payment API?
// Time box: 2 hours

// Hour 1: Read docs, get credentials
// Hour 2: Make a test charge

result = paymentAPI.charge({
    amount: 100,
    card: testCard,
    description: "Test"
})

// Outcome: Yes/No/Maybe with conditions
// Delete code, document findings
```

### Architectural Prototypes

Test system design:

```pseudocode
// Question: Can microservices communicate via message queue?

// Prototype: Simplest possible message flow

// Service A (producer)
function sendMessage() {
    queue.publish("orders", { orderId: 123 })
    print("Sent!")
}

// Service B (consumer)
function receiveMessage() {
    queue.subscribe("orders", message => {
        print("Received: " + message.orderId)
    })
}

// Run both, verify messages flow
// Answer: Yes, it works. Now design properly.
```

## What to Ignore in Prototypes

| Skip | Why |
|------|-----|
| Error handling | Clutters the experiment |
| Validation | Not testing that |
| Security | Prototype isn't production |
| Performance | Unless that's the question |
| Complete functionality | Only build what answers the question |
| Documentation | It's throwaway |
| Tests | Code is deleted anyway |

## Warning: Prototype Pitfalls

### The "Clean It Up Later" Trap

```pseudocode
// Manager: "Great prototype! Ship it."
// Developer: "But it's a prototype..."
// Manager: "Just clean it up a bit."

// DON'T DO THIS!
// Prototypes have fundamental shortcuts
// "Cleaning up" is harder than rewriting
```

### Make It Clear It's a Prototype

```pseudocode
// Name it obviously
prototype_payment_spike.py
THROW_AWAY_ui_test.html

// Or use a separate branch
git checkout -b prototype/payment-api

// Document it
// ⚠️ PROTOTYPE - DO NOT SHIP
// This code answers the question: Can we integrate with X?
// It ignores: security, errors, performance
// DELETE after review
```

### Set Time Limits

```
Spike: Payment API integration
Time Box: 4 hours
Question: Can we charge cards with this API?

Hour 0: Start
Hour 4: Stop, document, delete
```

## Post-It Note Architecture

For quick design exploration:

```
┌─────────┐     ┌─────────┐     ┌─────────┐
│  User   │────▶│   API   │────▶│  Order  │
│Interface│     │ Gateway │     │ Service │
└─────────┘     └─────────┘     └────┬────┘
                                     │
                                     ▼
┌─────────┐     ┌─────────┐     ┌─────────┐
│ Payment │◀────│  Event  │◀────│Inventory│
│ Service │     │   Bus   │     │ Service │
└─────────┘     └─────────┘     └─────────┘
```

Benefits:
- Fast to create and modify
- No code investment
- Easy to discuss and iterate
- Physical collaboration

## Summary

### When to Prototype

- Uncertain about feasibility
- New technology or approach
- Need stakeholder feedback early
- High-risk decisions
- Complex integration

### How to Prototype Well

1. Define the question clearly
2. Build the minimum to answer it
3. Skip everything irrelevant
4. Time-box the effort
5. Document the findings
6. **Throw away the code**

### The Golden Rule

> "If you find yourself working to 'clean up' prototype code, stop. That's a sign the prototype served its purpose. Now write production code from scratch with proper architecture."

---

*Based on concepts from "The Pragmatic Programmer" by David Thomas and Andrew Hunt.*
