# DDD Specification Pattern

## Definition

The Specification pattern encapsulates business rules as objects, transforming domain logic into reusable, composable predicates. Rather than scattering conditional logic throughout the codebase, specifications externalize rules into objects that can be combined, tested, and reasoned about independently.

## Purpose

- **Reusability**: Define business rules once, use everywhere
- **Composability**: Combine simple specs into complex ones (AND, OR, NOT)
- **Testability**: Test rules in isolation without domain object instantiation
- **Clarity**: Domain rules become explicit, first-class citizens in code
- **Flexibility**: Change business logic without modifying clients

## Core Concept

A specification is a predicate object that answers: "Does this entity satisfy this rule?"

```
Specification: isSatisfiedBy(candidate) → boolean
```

Specifications serve as bridges between:
- Domain knowledge (business rules)
- Technical needs (selection, validation, creation)

---

## Pseudocode: Specification Interface

```pseudocode
interface Specification<T>
    method isSatisfiedBy(candidate: T) → boolean
    method and(other: Specification<T>) → Specification<T>
    method or(other: Specification<T>) → Specification<T>
    method not() → Specification<T>
end interface
```

### Base Abstract Implementation

```pseudocode
abstract class AbstractSpecification<T> implements Specification<T>
    abstract method isSatisfiedBy(candidate: T) → boolean
    
    method and(other: Specification<T>) → Specification<T>
        return new CompositeSpecification(this, other, AND)
    end method
    
    method or(other: Specification<T>) → Specification<T>
        return new CompositeSpecification(this, other, OR)
    end method
    
    method not() → Specification<T>
        return new NegationSpecification(this)
    end method
end class
```

---

## Concrete Specification Examples

### Example 1: Customer Credit Limit Specification

```pseudocode
class CustomerWithGoodCreditSpecification extends AbstractSpecification<Customer>
    method isSatisfiedBy(customer: Customer) → boolean
        return customer.creditRating > 700 AND customer.yearsCustomer > 2
    end method
end class
```

### Example 2: Active Product Specification

```pseudocode
class ActiveProductSpecification extends AbstractSpecification<Product>
    field minStock: integer
    
    constructor(minStock: integer)
        this.minStock = minStock
    end constructor
    
    method isSatisfiedBy(product: Product) → boolean
        return product.isActive AND product.stock >= minStock
    end method
end class
```

### Example 3: Eligible for Discount Specification

```pseudocode
class EligibleForDiscountSpecification extends AbstractSpecification<Order>
    field minAmount: Money
    field maxDiscount: Percentage
    
    constructor(minAmount: Money, maxDiscount: Percentage)
        this.minAmount = minAmount
        this.maxDiscount = maxDiscount
    end constructor
    
    method isSatisfiedBy(order: Order) → boolean
        return order.total >= minAmount AND order.currentDiscount < maxDiscount
    end method
end class
```

---

## Composite Specifications: AND, OR, NOT

### AND Specification

```pseudocode
class AndSpecification<T> extends AbstractSpecification<T>
    field left: Specification<T>
    field right: Specification<T>
    
    constructor(left: Specification<T>, right: Specification<T>)
        this.left = left
        this.right = right
    end constructor
    
    method isSatisfiedBy(candidate: T) → boolean
        return left.isSatisfiedBy(candidate) AND right.isSatisfiedBy(candidate)
    end method
end class
```

**Usage:**
```pseudocode
goodCreditSpec = new CustomerWithGoodCreditSpecification()
longTermSpec = new CustomerWithLongHistorySpecification()
premiumSpec = goodCreditSpec.and(longTermSpec)

if premiumSpec.isSatisfiedBy(customer) then
    // Offer premium tier
end if
```

### OR Specification

```pseudocode
class OrSpecification<T> extends AbstractSpecification<T>
    field left: Specification<T>
    field right: Specification<T>
    
    constructor(left: Specification<T>, right: Specification<T>)
        this.left = left
        this.right = right
    end constructor
    
    method isSatisfiedBy(candidate: T) → boolean
        return left.isSatisfiedBy(candidate) OR right.isSatisfiedBy(candidate)
    end method
end class
```

**Usage:**
```pseudocode
platinumSpec = new PlatinumMemberSpecification()
vipSpec = new VipMemberSpecification()
eligibleSpec = platinumSpec.or(vipSpec)

if eligibleSpec.isSatisfiedBy(customer) then
    // Apply premium treatment
end if
```

### NOT Specification

```pseudocode
class NotSpecification<T> extends AbstractSpecification<T>
    field spec: Specification<T>
    
    constructor(spec: Specification<T>)
        this.spec = spec
    end constructor
    
    method isSatisfiedBy(candidate: T) → boolean
        return NOT spec.isSatisfiedBy(candidate)
    end method
end class
```

**Usage:**
```pseudocode
blacklistedSpec = new BlacklistedCustomerSpecification()
validSpec = blacklistedSpec.not()

if validSpec.isSatisfiedBy(customer) then
    // Process normally
end if
```

---

## Usage Patterns

### 1. Validation

Use specifications to validate entities conform to business rules before state changes.

```pseudocode
class Order
    field items: List<OrderItem>
    field customer: Customer
    
    method addItem(item: OrderItem) → void
        if OrderMinimumSpecification.isSatisfiedBy(this) then
            items.add(item)
        else
            throw InvalidOrderException("Order does not meet minimum requirements")
        end if
    end method
end class
```

### 2. Repository Selection

Use specifications to query entities matching domain conditions.

```pseudocode
interface CustomerRepository
    method findSatisfying(spec: Specification<Customer>) → List<Customer>
end interface

class SqlCustomerRepository implements CustomerRepository
    method findSatisfying(spec: Specification<Customer>) → List<Customer>
        results = executeQuery("SELECT * FROM customers")
        return results.filter(customer → spec.isSatisfiedBy(customer))
    end method
end class

// Usage
premiumCustomers = repository.findSatisfying(
    premiumSpec.and(geographicSpec).and(activeSpec)
)
```

### 3. Specification-Driven Creation

Use specifications to guide object construction.

```pseudocode
class OrderFactory
    method createOrder(customer: Customer, items: List<OrderItem>) → Order
        if NOT CustomerWithGoodCreditSpecification.isSatisfiedBy(customer) then
            throw InvalidCustomerException()
        end if
        
        if NOT MinimumOrderValueSpecification.isSatisfiedBy(items) then
            throw InvalidOrderException()
        end if
        
        order = new Order(customer, items)
        applyApplicableDiscounts(order)
        return order
    end method
    
    method applyApplicableDiscounts(order: Order) → void
        if EligibleForNewCustomerDiscount.isSatisfiedBy(order) then
            order.applyDiscount(newCustomerDiscount)
        end if
        
        if EligibleForBulkDiscount.isSatisfiedBy(order) then
            order.applyDiscount(bulkDiscount)
        end if
    end method
end class
```

### 4. Policy Application

Use specifications to apply business policies conditionally.

```pseudocode
class ShippingPolicyEngine
    field policies: List<(Order, Specification<Order>) Pair>
    
    method applyPolicies(order: Order) → void
        for each (policy, spec) in policies do
            if spec.isSatisfiedBy(order) then
                policy.apply(order)
            end if
        end for
    end method
end class
```

---

## Practical Business Examples

### E-Commerce: Customer Eligibility

```pseudocode
// Define individual rules
minPurchaseSpec = new MinimumPurchaseAmountSpecification(minimumAmount)
activeAccountSpec = new ActiveAccountSpecification()
noBannedSpec = new NotBannedSpecification()

// Compose into policy
standardCustomerPolicy = minPurchaseSpec.and(activeAccountSpec).and(noBannedSpec)

// Use for routing
if standardCustomerPolicy.isSatisfiedBy(customer) then
    assignToStandardShipping()
else
    assignToExpressShipping()
end if
```

### Banking: Loan Approval

```pseudocode
class LoanApprovalSpecification extends AbstractSpecification<LoanApplication>
    field creditChecker: Specification<Applicant>
    field incomeVerifier: Specification<Applicant>
    field debtRatioLimiter: Specification<Applicant>
    
    method isSatisfiedBy(application: LoanApplication) → boolean
        applicant = application.applicant
        return creditChecker.isSatisfiedBy(applicant) AND
               incomeVerifier.isSatisfiedBy(applicant) AND
               debtRatioLimiter.isSatisfiedBy(applicant)
    end method
end class
```

### SaaS: Feature Eligibility

```pseudocode
class AdvancedAnalyticsEligibleSpecification extends AbstractSpecification<Account>
    method isSatisfiedBy(account: Account) → boolean
        return account.plan >= PROFESSIONAL AND
               account.monthlyActiveUsers > 1000 AND
               account.accountAge >= 6 MONTHS
    end method
end class

// Use in feature flags
if analyticsSpec.isSatisfiedBy(account) then
    enableAdvancedAnalytics(account)
end if
```

---

## Benefits Table

| Benefit | Description | Example |
|---------|-------------|---------|
| **Encapsulation** | Business rules are objects, not scattered conditionals | `premiumCustomerSpec` vs 10 `if` statements |
| **Reusability** | Rules used across validation, queries, policies | One spec, three use cases |
| **Composability** | Build complex logic from simple specs | `spec1.and(spec2).or(spec3)` |
| **Testability** | Test rules without domain object context | Test spec in isolation |
| **Clarity** | Domain rules become explicit and named | Domain language in code |
| **Maintainability** | Business logic changes in one place | Modify spec, all uses updated |
| **Flexibility** | Easy to add new specifications | New rule = new class |
| **Queryability** | Rules inform database queries | Repository finds matching entities |

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Pattern Name** | Specification |
| **Intent** | Encapsulate business rules as composable objects |
| **Participants** | Specification (interface), Concrete Specs, Composite Specs |
| **Collaborators** | Domain Objects, Repositories, Factories, Policies |
| **Best For** | Complex validation, selection, policy application |
| **Key Methods** | `isSatisfiedBy(candidate)`, `and()`, `or()`, `not()` |
| **Typical Cost** | Slight verbosity; high clarity payoff |
| **Related Patterns** | Composite, Strategy, Repository |
| **Antipattern Addresses** | Logic scattered across codebase; hard-to-test rules |

---

## Key Takeaways

1. **Specifications = Predicates as Objects**: Transform "does X satisfy Y?" into an object that can be composed and reused.

2. **Composition Over Duplication**: Build complex rules by combining simple, focused specifications.

3. **Domain Knowledge, First-Class**: Business rules become explicit, testable, nameable entities in the codebase.

4. **Bridge Pattern**: Specifications connect domain knowledge (business) with technical needs (databases, validation, policies).

5. **Works Best With**: Repository pattern (for querying), Factory pattern (for construction), Policy pattern (for conditional application).

---

*Based on concepts from "Domain-Driven Design" by Eric Evans.*
