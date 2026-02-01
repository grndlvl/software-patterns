# Ubiquitous Language

## Definition

**Ubiquitous Language** is a shared, precise vocabulary used consistently by all team members—developers, domain experts, product owners, and stakeholders—when discussing the domain model. This language is reflected in both conversations and code, creating a direct mapping between domain concepts and their software representations.

The language is "ubiquitous" because it pervades all aspects of the project:
- Requirements documents
- User stories and acceptance criteria
- Code (class names, method names, variable names)
- Tests and test scenarios
- Team discussions and meetings
- Documentation

## Why It Matters

### Eliminates Translation Overhead

Without a Ubiquitous Language, teams suffer from constant translation:
- Domain experts describe concepts in business terms
- Developers translate to technical terms in code
- During discussions, concepts get translated back and forth
- Information is lost, misunderstandings accumulate

With Ubiquitous Language:
- Everyone speaks the same language
- Code directly reflects domain concepts
- No mental translation required
- Communication becomes precise and efficient

### Catches Misunderstandings Early

When domain experts review code or discuss features using the actual code terminology, mismatches between their understanding and the implementation become immediately obvious. If a domain expert says "that's not how we think about it," you've caught a modeling error before it becomes expensive.

### Creates Living Documentation

Code written with Ubiquitous Language documents itself. A method named `calculatePremiumForRiskCategory()` is self-explanatory to anyone who knows the domain, whereas `computeAmount()` requires detective work.

### Supports Model Evolution

As the team's understanding deepens, the language evolves. New insights trigger refactoring to align code with the improved model, keeping the codebase aligned with current domain understanding.

## Building the Language

### Start with Domain Expert Conversations

The language emerges from collaborative exploration:

1. **Listen for nouns** - These become entities, value objects, or aggregates
2. **Listen for verbs** - These become methods, domain events, or processes
3. **Listen for qualifiers** - These become attributes, enums, or specifications
4. **Probe ambiguities** - When domain experts use multiple terms for the same concept, dig deeper
5. **Challenge technical jargon** - Replace generic programming terms with domain-specific alternatives

### Document Key Terms

Maintain a glossary of domain terms with precise definitions. This isn't just for newcomers—it's a negotiation tool for the team to agree on exact meanings.

Example glossary entries:

| Term | Definition | Example |
|------|------------|---------|
| **Premium** | The amount a customer pays for insurance coverage during a specific period | Monthly premium: $150 |
| **Risk Category** | Classification of insurable items based on loss probability (Low, Medium, High, Critical) | A 2020 sedan in good condition is "Low" risk |
| **Coverage Period** | The time span during which an insurance policy is active | Jan 1, 2026 - Dec 31, 2026 |
| **Claim** | A formal request by a policyholder for payment of benefits under an insurance policy | Claim #2026-001234 for collision damage |
| **Underwriting** | The process of evaluating risk and determining coverage terms and pricing | Underwriting approved with standard premium |

### Let Language Emerge from Modeling

Don't force the language prematurely. As you model scenarios through Event Storming, domain storytelling, or example mapping, the language naturally emerges. Capture it, refine it, and codify it.

## Language in Code

### Class and Type Names

Use domain terms directly:

```pseudocode
// Good - Ubiquitous Language
class InsurancePolicy {
  policyNumber: PolicyNumber
  coveragePeriod: CoveragePeriod
  premium: Money
  riskCategory: RiskCategory
}

class PolicyNumber {
  value: string
}

enum RiskCategory {
  LOW, MEDIUM, HIGH, CRITICAL
}

// Bad - Generic technical terms
class Policy {
  id: string
  startDate: Date
  endDate: Date
  amount: Decimal
  category: int
}
```

### Method Names

Methods should read like domain processes:

```pseudocode
class InsurancePolicy {
  // Good - expresses domain intent
  function renewForAnotherYear(): InsurancePolicy
  function adjustPremiumForRiskChange(newCategory: RiskCategory): void
  function processClaim(claim: Claim): ClaimDecision
  function cancelWithRefund(reason: CancellationReason): Refund

  // Bad - generic CRUD operations
  function update(): void
  function calculate(): Decimal
  function process(): Result
  function handleRequest(): Response
}
```

### Variable and Parameter Names

Even local variables should use domain language:

```pseudocode
// Good
function calculateAnnualPremium(policy: InsurancePolicy): Money {
  baseRate = rateTable.getRateFor(policy.riskCategory)
  discountFactor = policy.loyaltyDiscount.asMultiplier()
  return baseRate.multiply(discountFactor)
}

// Bad
function calculate(p: Policy): Decimal {
  rate = table.get(p.category)
  factor = p.discount.convert()
  return rate * factor
}
```

### Domain Events

Events capture what happened in domain terms:

```pseudocode
// Good - domain language
class PolicyRenewed {
  policyNumber: PolicyNumber
  renewalDate: Date
  newCoveragePeriod: CoveragePeriod
}

class PremiumAdjusted {
  policyNumber: PolicyNumber
  oldPremium: Money
  newPremium: Money
  reason: AdjustmentReason
}

// Bad - technical language
class PolicyUpdated {
  id: string
  timestamp: long
  changes: Map<string, any>
}
```

## Evolving the Language

### Recognize Breakthrough Moments

Sometimes a conversation reveals that the team has been thinking about a concept incorrectly. This is a **modeling breakthrough**. When it happens:

1. **Update the glossary** - Capture the new understanding
2. **Refactor the code** - Align implementation with new insight
3. **Share with the team** - Ensure everyone adopts the refined language

Example breakthrough:
> **Before**: We thought "discount" was just a percentage off the premium
> **After**: We realized there are multiple discount types—loyalty, multi-policy, low-risk—each calculated differently and with different business rules

This leads to refactoring from:

```pseudocode
// Before
class Policy {
  discountPercentage: Decimal
}

// After
class Policy {
  discounts: DiscountSet
}

interface Discount {
  function calculateFor(policy: Policy): Money
}

class LoyaltyDiscount implements Discount {
  yearsWithCompany: int
  function calculateFor(policy: Policy): Money
}

class MultiPolicyDiscount implements Discount {
  numberOfPolicies: int
  function calculateFor(policy: Policy): Money
}
```

### Keep Language Consistent Across Contexts

In large systems, different Bounded Contexts may use the same term differently. That's fine—just be explicit about context boundaries:

```pseudocode
// In Sales Context
class Customer {
  function requestQuote(vehicle: Vehicle): Quote
}

// In Underwriting Context
class Applicant {  // Not "Customer" - different stage of lifecycle
  function assessRisk(): RiskAssessment
}

// In Billing Context
class Policyholder {  // Not "Customer" - different responsibilities
  function processPayment(amount: Money): PaymentResult
}
```

## Common Pitfalls

### Pitfall: Letting Technical Terms Dominate

**Symptom**: Code full of `Manager`, `Handler`, `Processor`, `Service`, `Utility`

**Fix**: Challenge every technical term. Ask "What does this represent in the domain?" Replace `InvoiceManager` with `BillingCalculator` or `InvoiceRepository`.

### Pitfall: Using Ambiguous Terms

**Symptom**: Team members have different understandings of key concepts

**Fix**: When you notice ambiguity (e.g., does "active policy" mean "currently in coverage period" or "not canceled"?), stop and clarify. Document the agreed-upon meaning.

### Pitfall: Language Drift

**Symptom**: Code uses old terminology while domain experts have moved on

**Fix**: Regularly review code with domain experts. When language evolves, schedule refactoring to keep code aligned.

### Pitfall: One-Size-Fits-All Language

**Symptom**: Trying to force the same terms across all contexts in a large system

**Fix**: Recognize that different Bounded Contexts may need different languages. Sales talks about "leads," while Underwriting talks about "applicants," and Billing talks about "policyholders." That's healthy segregation.

## Pseudocode Examples Showing Naming

### Example 1: Order Fulfillment

```pseudocode
class Order {
  orderNumber: OrderNumber
  customer: CustomerId
  items: List<OrderItem>
  shippingAddress: Address
  status: OrderStatus

  function placeOrder(): void
  function addItem(product: Product, quantity: Quantity): void
  function removeItem(lineItemId: LineItemId): void
  function ship(carrier: Carrier, trackingNumber: TrackingNumber): void
  function markAsDelivered(): void
  function initiateReturn(reason: ReturnReason): Return
}

enum OrderStatus {
  DRAFT, PLACED, CONFIRMED, SHIPPED, DELIVERED, RETURNED
}

class OrderItem {
  lineItemId: LineItemId
  product: Product
  quantity: Quantity
  unitPrice: Money

  function calculateSubtotal(): Money
}

class ShippingStrategy {
  function calculateShippingCost(order: Order): Money
  function estimateDeliveryDate(order: Order): Date
}
```

### Example 2: Loan Approval

```pseudocode
class LoanApplication {
  applicationId: ApplicationId
  applicant: Applicant
  requestedAmount: Money
  purpose: LoanPurpose
  status: ApplicationStatus

  function submit(): void
  function performCreditCheck(): CreditScore
  function assessAffordability(): AffordabilityResult
  function approve(terms: LoanTerms): ApprovedLoan
  function reject(reason: RejectionReason): void
  function requestMoreInformation(requirements: List<Requirement>): void
}

class Applicant {
  name: PersonName
  income: AnnualIncome
  employmentStatus: EmploymentStatus
  creditHistory: CreditHistory
}

class AffordabilityResult {
  monthlyIncome: Money
  existingObligations: Money
  proposedMonthlyPayment: Money
  debtToIncomeRatio: Percentage

  function isAffordable(): boolean
}

enum ApplicationStatus {
  DRAFT, SUBMITTED, UNDER_REVIEW, APPROVED, REJECTED, MORE_INFO_NEEDED
}
```

### Example 3: Inventory Management

```pseudocode
class InventoryItem {
  sku: StockKeepingUnit
  product: Product
  quantityOnHand: Quantity
  reorderPoint: Quantity
  reorderQuantity: Quantity
  warehouse: WarehouseLocation

  function allocateForOrder(quantity: Quantity): Allocation
  function receive(shipment: InboundShipment): void
  function adjustForDamage(quantity: Quantity, reason: AdjustmentReason): void
  function checkIfReorderNeeded(): boolean
  function createReplenishmentOrder(): ReplenishmentOrder
}

class Allocation {
  itemSku: StockKeepingUnit
  allocatedQuantity: Quantity
  reservationExpiry: DateTime

  function confirm(): void
  function release(): void
}

class InboundShipment {
  shipmentId: ShipmentId
  supplier: Supplier
  items: List<ShipmentItem>
  receivedDate: Date

  function recordReceipt(): void
  function performQualityCheck(): QualityCheckResult
}
```

## Summary Table

| Aspect | With Ubiquitous Language | Without Ubiquitous Language |
|--------|--------------------------|------------------------------|
| **Communication** | Precise, no translation needed | Constant back-and-forth translation |
| **Code Readability** | Self-documenting, domain experts can read it | Requires technical knowledge to understand |
| **Model Alignment** | Code reflects domain understanding | Disconnect between domain and implementation |
| **Onboarding** | New members learn domain through code | New members must learn two languages |
| **Evolution** | Language evolves with understanding | Terminology becomes stale and misleading |
| **Bug Detection** | Domain experts catch errors in reviews | Domain errors hide behind technical abstractions |
| **Requirements** | Clear, unambiguous specifications | Ambiguous, subject to interpretation |
| **Maintenance** | Changes follow domain logic naturally | Changes require mental translation |

---

*Based on concepts from "Domain-Driven Design" by Eric Evans.*
