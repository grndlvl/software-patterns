# Estimating

## Definition

> "Estimate to avoid surprises."
> — Dave Thomas & Andy Hunt

Good estimates help everyone plan. Bad estimates create frustration, missed deadlines, and broken trust. Learn to estimate well.

## The Purpose of Estimates

Estimates answer the question: **"How long will this take?"**

But the real question behind that is often:
- "Can we make the deadline?"
- "Should we do this project?"
- "How do we plan our resources?"

Understanding the *why* helps you give better answers.

## Units Matter

Choose units that reflect your confidence:

| Duration | Unit | Implies |
|----------|------|---------|
| 1-15 days | Days | Fairly confident |
| 3-6 weeks | Weeks | Some uncertainty |
| 2-6 months | Months | Significant uncertainty |
| More | "I need to break this down" | Too uncertain to estimate |

```pseudocode
// Bad: "It'll take 123 hours"
// Implies precision you don't have

// Good: "About 3 weeks"
// Honest uncertainty
```

## Where Estimates Come From

### 1. Ask Someone Who's Done It

```pseudocode
// Best source: Experience

you: "How long to add OAuth?"
colleague: "I did it last year. Basic setup is 2 days.
           Edge cases and testing took another week.
           Budget 2 weeks to be safe."
```

### 2. Break It Down

```pseudocode
// Task: Build user registration

// Break into subtasks:
// - Design database schema        (0.5 days)
// - Create API endpoint           (1 day)
// - Build form UI                 (1 day)
// - Add validation                (0.5 days)
// - Write tests                   (1 day)
// - Handle errors                 (0.5 days)
// - Documentation                 (0.5 days)
// Total: 5 days
// Add buffer: 7 days (~1.5 weeks)
```

### 3. Build a Model

For complex estimates, identify the factors:

```pseudocode
// Estimating data migration

factors:
  - Number of records: 1,000,000
  - Processing time per record: 100ms
  - Batch size: 1,000
  - Network latency: 50ms/batch

calculation:
  batches = 1,000,000 / 1,000 = 1,000 batches
  processing = 1,000,000 * 0.1s = 100,000s
  network = 1,000 * 0.05s = 50s
  total = ~28 hours

// Add buffer for issues: 2 days
```

### 4. Iterate with Feedback

Track your estimates vs. actuals:

```
| Task             | Estimated | Actual | Ratio |
|------------------|-----------|--------|-------|
| User auth        | 3 days    | 5 days | 1.7x  |
| Payment API      | 1 week    | 2 weeks| 2.0x  |
| Report generator | 2 weeks   | 2 weeks| 1.0x  |

Average ratio: 1.6x
Apply to future estimates
```

## The Estimation Process

### Step 1: Understand the Ask

```pseudocode
// Don't estimate immediately

// Ask:
// - What exactly is needed?
// - What's the scope?
// - What quality level?
// - Are there existing constraints?
// - What's driving the deadline?
```

### Step 2: Build a Model of the System

```pseudocode
// Mental model of what's involved

user_registration:
  ├── frontend:
  │   ├── form component
  │   ├── validation UI
  │   └── success/error states
  ├── backend:
  │   ├── API endpoint
  │   ├── validation logic
  │   ├── database operations
  │   └── email verification
  └── infrastructure:
      ├── email service setup
      └── database migrations
```

### Step 3: Break into Components

```pseudocode
// Estimate each component

components = [
    ("Form component", "1 day"),
    ("API endpoint", "0.5 days"),
    ("Validation", "0.5 days"),
    ("Database", "0.5 days"),
    ("Email service", "1 day"),
    ("Testing", "1 day"),
    ("Buffer", "1 day")
]

total = sum(components)  // 5.5 days ≈ 1 week
```

### Step 4: Give Ranges

```pseudocode
// Single number: "5 days"
// Problem: Treated as commitment

// Range: "4-8 days, most likely 5-6"
// Better: Shows uncertainty

// Confidence: "80% confident it's under 2 weeks"
// Best: Explicitly states confidence
```

## PERT Estimation

**P**rogram **E**valuation and **R**eview **T**echnique:

```pseudocode
// Three-point estimate
optimistic = 3    // Best case
mostLikely = 5    // Typical case
pessimistic = 12  // Worst case

// Weighted average
expected = (optimistic + 4*mostLikely + pessimistic) / 6
         = (3 + 20 + 12) / 6
         = 5.8 days
```

## Common Estimation Mistakes

### 1. Anchoring

```pseudocode
// Bad: Manager says "This should take a day, right?"
// You think: "Well... maybe... I guess?"

// Good: "Let me think about it and get back to you."
// Then estimate independently
```

### 2. Forgetting Tasks

```pseudocode
// Often forgotten:
// - Testing
// - Code review
// - Documentation
// - Deployment
// - Bug fixes
// - Meetings
// - Context switching
// - Learning curve
```

### 3. Optimism Bias

```pseudocode
// Thought: "If everything goes perfectly..."
// Reality: Everything never goes perfectly

// Add buffer:
// - Small task: +20%
// - Medium task: +50%
// - Large task: +100%
// - Unknown tech: +200%
```

### 4. Precision Theater

```pseudocode
// Bad: "It will take 17.5 hours"
// Implies false precision

// Good: "About 2-3 days"
// Honest about uncertainty
```

## Saying "I Don't Know"

It's OK to not have an estimate:

```pseudocode
// Good responses:
"I need to research this first. Give me a day."
"I've never done this. Let me do a spike."
"This is too big. Can we break it down?"
"I can estimate the first phase, but not the rest yet."
```

## When Asked for Immediate Estimates

```pseudocode
// Pressure: "Quick, how long?"

// Options:
1. "Off the top of my head, maybe X, but let me verify."
2. "I'd need to look at the code first."
3. "That sounds like a week or two, but I'm not confident."
4. "Can I get back to you in an hour?"
```

## Improving Over Time

### Keep a Log

```
| Date | Task | Estimate | Actual | Notes |
|------|------|----------|--------|-------|
| 1/15 | API  | 3 days   | 5 days | Auth was harder |
| 1/22 | UI   | 1 week   | 1 week | On track |
| 2/01 | DB   | 2 days   | 1 day  | Overestimated |
```

### Review and Adjust

```pseudocode
// Monthly review
function reviewEstimates() {
    accuracy = actual / estimated

    if accuracy > 1.5 {
        // Consistently underestimating
        // Increase future estimates
    } else if accuracy < 0.8 {
        // Overestimating
        // Decrease or you're sandbagging
    }
}
```

## Summary

| Do | Don't |
|----|-------|
| Ask clarifying questions | Estimate immediately |
| Break tasks down | Give one big number |
| Give ranges | Imply false precision |
| Include buffer | Assume best case |
| Track actuals | Forget to learn |
| Say "I don't know" | Make up numbers |
| Use appropriate units | Say "17.5 hours" |

---

*Based on concepts from "The Pragmatic Programmer" by David Thomas and Andrew Hunt.*
