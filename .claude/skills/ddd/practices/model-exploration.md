# Model Exploration in Domain-Driven Design

## Definition

**Model Exploration** is the iterative, collaborative process of discovering, understanding, and refining the domain model through deep engagement with domain experts. It's not a one-time activity but a continuous cycle of learning, modeling, and validation that evolves as both domain understanding and implementation mature.

The goal is to uncover the **Ubiquitous Language** and the **core concepts** that truly matter in the domain, filtering out accidental complexity and surface-level details.

---

## Core Techniques

### 1. Domain Expert Interviews

Structured conversations with subject matter experts to capture domain knowledge.

**Best Practices:**
- Ask open-ended questions before narrow ones
- Listen for repeated concepts and terminology
- Ask "why" questions to understand business rules
- Record and review conversations
- Involve multiple experts (they often disagree—those disagreements are gold)

**Interview Structure:**
```
1. Context & History
   - How did the domain evolve?
   - What problems are you solving?
   - What's broken with current approaches?

2. Core Workflows
   - Walk me through a typical day
   - What's a successful outcome?
   - What prevents success?

3. Rules & Constraints
   - What are the non-negotiable rules?
   - Where do exceptions occur?
   - How are disputes resolved?

4. Terminology
   - What do you call [concept]?
   - When do you use that term vs. this one?
   - What's jargon vs. precise terminology?
```

### 2. Scenario Walkthroughs

Act out real or hypothetical scenarios from the domain to surface hidden behaviors.

**Example Scenario (E-Commerce Domain):**
```
Expert: "When a customer orders 10 units but we only have 7 in stock..."

Engineer: "What happens then?"

Expert: "Well, it depends. If it's a rush order from our VIP account, 
we'll backorder the 3. For regular customers, we typically 
suggest alternatives or cancel the order."

Engineer: "So the rule isn't 'check inventory'—it's 'apply fulfillment 
policy based on customer status and item availability'?"

Expert: "Exactly. And if it's a made-to-order item, backorders 
are always accepted."
```

**Walkthrough Checklist:**
- Start with happy path
- Explore edge cases systematically
- Ask "what if" for each decision point
- Validate with domain expert that behavior matches reality
- Identify which rules are truly invariant vs. configurable

### 3. CRC Cards (Class-Responsibility-Collaboration)

Low-tech, high-engagement modeling tool for discovering domain objects.

**Setup:**
```
┌─────────────────────────────────┐
│         Order                   │
├─────────────────────────────────┤
│ Responsibilities:               │
│ • Know its items                │
│ • Know its status               │
│ • Validate fulfillment rules    │
│                                 │
│ Collaborators:                  │
│ • LineItem                      │
│ • FulfillmentPolicy             │
│ • Customer                      │
└─────────────────────────────────┘
```

**Why CRC Cards Work:**
- Physical cards force conversations (can't hide in menus)
- Moving cards reveals collaboration patterns
- Low friction for iteration (erase and rewrite)
- Non-developers can participate as peers
- Natural springboard for "what if" scenarios

**CRC Session Flow:**
1. Identify primary entity (Order, Invoice, Shipment)
2. List its responsibilities (what it knows, what it decides)
3. Identify collaborators (what other objects help)
4. Act out a scenario with the cards
5. When blocked, split into new cards or merge existing ones
6. Repeat until scenario flows naturally

---

## The Whirlpool Process

Model exploration follows a cyclical, deepening pattern:

```
┌─────────────────────────────────────────────────────────┐
│                    Modeling                             │
│  • Extract language from expert interviews              │
│  • Sketch initial concepts and relationships            │
│  • Build CRC cards or domain diagrams                   │
└──────────────────┬──────────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────────┐
│                  Distillation                           │
│  • What are the core concepts?                          │
│  • What's accidental complexity?                        │
│  • How do concepts relate?                              │
│  • Can we simplify the language?                        │
└──────────────────┬──────────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────────┐
│                 Implementation                          │
│  • Write code that reflects the model                   │
│  • Use Ubiquitous Language in code                      │
│  • Test against real scenarios                          │
│  • Let gaps surface new questions                       │
└──────────────────┬──────────────────────────────────────┘
                   ↓
                (Cycle repeats with deeper understanding)
```

**Key Insight:** Each cycle should deepen understanding. Distillation is where the real work happens—filtering noise from signal.

---

## Questions to Ask Domain Experts

### About Core Processes

- What are the main workflows? (step by step)
- Which steps are critical? Which can be delayed?
- What causes a workflow to fail or restart?
- How do you know when a process is complete?
- What manual steps can't be automated?

### About Rules & Constraints

- What are the unbreakable rules? (laws, regulations, business policy)
- When do exceptions occur? How are they handled?
- What happens if we break rule X?
- Which rules apply all the time vs. sometimes?
- Are there conflicting rules? How are ties broken?

### About Language

- What do you call [concept]? Is there a formal term?
- Are "customer" and "account" the same thing or different?
- When you say "process," do you always mean the same thing?
- Do different teams use different words for the same thing?
- What terminology would be wrong or misleading?

### About Data & State

- What information must we always know about [entity]?
- How does [entity] change over time?
- What information is historical vs. current?
- Can [entity] go back to a previous state?
- When we need to report, what data do we actually use?

### About Variability

- Does this behavior change based on [customer type/geography/product]?
- Are there seasonal variations?
- Has this rule changed over time? Is it changing now?
- If we launched a new market, what would change?

---

## Signs of Good Model vs. Bad Model

| Aspect | Good Model | Bad Model |
|--------|-----------|-----------|
| **Language** | Experts naturally use terms from the model | Model uses technical jargon; experts use different words |
| **Completeness** | Handles real scenarios without stretching | Edge cases require special cases; scenario walkthroughs fail |
| **Relationships** | Objects collaborate naturally; low coupling | Objects need information from many others; high coupling |
| **Invariants** | Business rules are obvious in code | Rules are scattered; easy to violate accidentally |
| **Simplicity** | Model has 4-7 core concepts | 20+ concepts; unclear boundaries |
| **Stability** | Model survives new requirements with small changes | New requirements require model redesign |
| **Communication** | Non-developers can follow the model | Model confuses domain experts |
| **Testability** | Business scenarios map directly to tests | Tests are implementation-focused, not business-focused |
| **Responsibility** | Each concept has clear, focused purpose | Bloated entities doing multiple unrelated things |
| **Generality** | Model is specific to the domain (not generic patterns) | Model applies everywhere; has no domain opinion |

---

## Refactoring Toward Deeper Insight

As understanding deepens, the model evolves. This process is called **refactoring toward deeper insight**.

### Pattern: Introduce a New Concept

**Initial Model (Shallow):**
```pseudocode
class Order:
  items: LineItem[]
  status: string  // "pending", "confirmed", "shipped", "delivered"
  
  canShip():
    if status == "confirmed" and hasInventory():
      status = "shipped"
      return true
    return false
```

**Problem:** Status is a magic string. Shipping logic is hidden in a method.

**Deeper Model (Insight):**
```pseudocode
class Order:
  items: LineItem[]
  fulfillmentState: FulfillmentState
  
  requestShipment():
    fulfillmentState.requestShipment(this)

interface FulfillmentState:
  requestShipment(order: Order): void

class ConfirmedState implements FulfillmentState:
  requestShipment(order: Order):
    if order.hasInventory():
      order.fulfillmentState = new ShippedState()
    else:
      // raise exception—can't ship from confirmed state
      throw CannotShipFromConfirmedState()

class PendingState implements FulfillmentState:
  requestShipment(order: Order):
    throw InvalidStateTransition("Cannot ship pending orders")
```

**What We Learned:**
- States aren't just labels—they're behavioral contracts
- "Can ship?" isn't a question; it's a business rule
- The model now **enforces** invariants, not just represents them

### Pattern: Extract Hidden Concepts

**Initial Model (Shallow):**
```pseudocode
class Payment:
  amount: decimal
  currency: string
  processor: string
  
  process():
    // 40 lines handling different processor APIs
    if processor == "stripe":
      // Stripe-specific logic
    else if processor == "paypal":
      // PayPal-specific logic
```

**Problem:** Domain concept (payment processing strategy) hidden in conditional logic.

**Deeper Model (Insight):**
```pseudocode
class Payment:
  amount: Money  // encapsulates amount + currency
  strategy: PaymentStrategy
  
  process():
    return strategy.authorize(amount)

interface PaymentStrategy:
  authorize(amount: Money): PaymentResult

class StripePaymentStrategy implements PaymentStrategy:
  authorize(amount: Money): PaymentResult:
    // Stripe-specific implementation

class PayPalPaymentStrategy implements PaymentStrategy:
  authorize(amount: Money): PaymentResult:
    // PayPal-specific implementation

class Money:
  amount: decimal
  currency: string
  
  add(other: Money): Money:
    if currency != other.currency:
      throw CurrencyMismatch()
    return Money(amount + other.amount, currency)
```

**What We Learned:**
- Money is a concept, not a primitive
- Payment strategies are explicit, not hidden
- Currency rules are enforced where they matter
- Model is open to new strategies without changing Payment

### Pattern: Capture Meaning in Type

**Initial Model (Shallow):**
```pseudocode
class Customer:
  accountNumber: string
  riskScore: int
  
  canApproveCredit():
    return riskScore > 700
```

**Problem:** What does riskScore of 700 mean? Where does that number come from?

**Deeper Model (Insight):**
```pseudocode
class Customer:
  accountNumber: AccountNumber
  riskAssessment: RiskAssessment
  
  canApproveCredit(): bool:
    return riskAssessment.isAcceptableRisk()

class RiskAssessment:
  score: CreditScore
  assessedAt: LocalDateTime
  
  isAcceptableRisk(): bool:
    return score >= CreditScore.MINIMUM_APPROVAL_THRESHOLD

class CreditScore:
  value: int  // 300-850 (FICO range, domain knowledge embedded)
  
  constructor(value: int):
    if value < 300 or value > 850:
      throw InvalidCreditScore()
  
  static MINIMUM_APPROVAL_THRESHOLD = CreditScore(700)
```

**What We Learned:**
- CreditScore has domain meaning (300-850)
- Thresholds are named, not magic numbers
- Type system now prevents invalid scores at creation time

---

## Model Exploration Checklist

Use this during and after exploration sessions:

**Language Alignment**
- [ ] Each core concept has a name domain experts recognize
- [ ] The same concept isn't called multiple things in the model
- [ ] No model terms are unknown to domain experts
- [ ] Experts nod in recognition when hearing the model described

**Completeness**
- [ ] Real scenarios can be walked through the model
- [ ] Edge cases have explicit handling (not "we'll handle that later")
- [ ] The model answers "what goes wrong?" questions
- [ ] State transitions are clear and enforced

**Simplicity**
- [ ] Core model has fewer than 10 primary concepts
- [ ] Each concept has a focused, single responsibility
- [ ] Relationships are clear and limited
- [ ] No artificial groupings or god objects

**Correctness**
- [ ] Invariants are explicitly enforced
- [ ] Invalid states are impossible (not just detected)
- [ ] Business rules are expressed as code, not comments
- [ ] Violations fail fast with clear errors

**Stability**
- [ ] Model survives several "what if" scenarios
- [ ] New rules extend the model, don't reshape it
- [ ] Adding a new customer type doesn't require rewriting core logic
- [ ] Past requirements are still supported

---

## Summary Table: Model Exploration Activities

| Activity | Duration | Participants | Output | When to Do It |
|----------|----------|--------------|--------|--------------|
| **Expert Interview** | 60-90 min | 1-2 experts, 2-3 engineers | Recording, notes, questions | Kicking off domain; discovery phase |
| **CRC Card Session** | 90-120 min | 3-5 engineers, 1 expert | Card descriptions, scenarios validated | Modeling core concepts |
| **Scenario Walkthrough** | 30-60 min | 2-3 engineers, 1-2 experts | Behavior documentation, gaps identified | Testing model completeness |
| **Language Refinement** | 45 min | Whole team | Updated glossary, term definitions | Weekly during development |
| **Refactoring Discussion** | 60 min | 2-3 engineers, 1 expert | Deeper model, new concepts introduced | When current model feels shallow |
| **Model Review** | 60 min | Whole team, 1 expert | Validated model, known gaps | End of iteration |

---

## Summary

Model Exploration is **not** a phase you complete and move on from. It's the **heartbeat of DDD**. 

As you build, as you test, as you fail—you learn more about the domain. Each learning refines the model. The best models aren't perfect at the start; they're **continuously distilled** through the whirlpool of modeling, distillation, and implementation.

The signs of a healthy modeling process:
1. Domain experts understand and recognize the model
2. Code reflects domain language naturally
3. New requirements extend rather than reshape the model
4. Invalid states are impossible, not just unlikely
5. Business rules are explicit, not hidden in conditionals

Keep asking: **"What does the domain actually need?"** Not "what pattern should we use?" or "what's most flexible?" The model should be **opinionated about the domain**, not generic.

---

*Based on concepts from "Domain-Driven Design" by Eric Evans.*
