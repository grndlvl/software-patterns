# Refactoring

> "Don't live with broken windows."
> — David Thomas & Andrew Hunt

## What is Refactoring?

**Refactoring** is the disciplined technique of restructuring existing code without changing its external behavior. It's about improving the internal structure, readability, and maintainability while keeping all tests passing.

Refactoring is NOT:
- Rewriting from scratch
- Adding new features
- Fixing bugs (though it often reveals them)

Refactoring IS:
- Continuous improvement
- Reducing technical debt
- Making code easier to understand
- Preparing code for new features

## When to Refactor

### Code Smells (Signals to Refactor)

| Smell | Description | Example |
|-------|-------------|---------|
| **Duplication** | Same code/logic appears multiple times | Copy-pasted validation logic |
| **Long Functions** | Function doing too many things | 200-line function with 5 responsibilities |
| **Large Classes** | Class with too many responsibilities | "God object" managing everything |
| **Long Parameter Lists** | Function taking 5+ parameters | `createUser(name, email, age, address, phone, role, status)` |
| **Divergent Change** | One class changes for multiple reasons | Class modified for DB, UI, and business logic |
| **Shotgun Surgery** | One change requires edits in many places | Changing date format touches 20 files |
| **Feature Envy** | Method uses another class's data more than its own | Method calling 10 getters on another object |
| **Data Clumps** | Same group of data appearing together | `(x, y, z)` coordinates always passed together |
| **Primitive Obsession** | Using primitives instead of small objects | Using strings for phone numbers, emails |
| **Switch Statements** | Type code with switch/case logic | `switch(type)` instead of polymorphism |
| **Comments** | Excessive comments explaining what code does | Code so complex it needs paragraph explanations |

### The Rule of Three

> "Three strikes and you refactor."

1. **First time:** Write it
2. **Second time:** Wince at duplication, but duplicate anyway
3. **Third time:** Refactor

## How to Refactor Safely

### The Golden Rule: Tests First

**Never refactor without a safety net.**

```pseudocode
// BEFORE refactoring anything
IF no_tests_exist THEN
    write_characterization_tests()
END IF

VERIFY all_tests_pass()

// NOW refactor in small steps
refactor_one_thing()
run_tests()
VERIFY all_tests_pass()

refactor_next_thing()
run_tests()
VERIFY all_tests_pass()

// Repeat until done
```

### Small Steps Protocol

| DO | DON'T |
|----|-------|
| Change one thing at a time | Refactor multiple patterns simultaneously |
| Run tests after each micro-change | Wait until "done" to test |
| Commit after each successful refactor | Make huge commits with mixed changes |
| Keep builds green | Leave broken code "for later" |
| Use automated refactoring tools | Manually edit 50 files |

## Martin Fowler's Refactoring Catalog (Overview)

### Composing Methods

- **Extract Method** - Turn code fragment into its own method
- **Inline Method** - Replace method call with method body (when too simple)
- **Extract Variable** - Put expression result in a self-explanatory variable
- **Replace Temp with Query** - Extract expression into method

### Moving Features

- **Move Method** - Move method to class where it's more used
- **Move Field** - Move field to class where it's more used
- **Extract Class** - Create new class for subset of responsibilities
- **Inline Class** - Merge class into another when it does too little

### Organizing Data

- **Replace Magic Number with Named Constant**
- **Encapsulate Field** - Make field private, add accessors
- **Replace Type Code with Class** - Turn coded type into class

### Simplifying Conditionals

- **Decompose Conditional** - Extract condition and branches into methods
- **Consolidate Conditional** - Combine related conditionals
- **Replace Conditional with Polymorphism** - Use inheritance instead of switch

## Real-Time Refactoring vs. Scheduled Refactoring

### Real-Time (The Pragmatic Way)

**"Leave the code better than you found it."** — The Scout Rule

```pseudocode
// You're adding a feature
FUNCTION add_new_feature()
    // See duplication or mess while working
    IF code_smell_detected THEN
        refactor_immediately()  // Takes 5 minutes
        THEN add_feature()      // Now easier to add
    ELSE
        add_feature()
    END IF
END FUNCTION
```

**Benefits:**
- No separate refactoring phase needed
- Context is fresh in your mind
- Continuous improvement
- No "refactoring sprints"

### Scheduled Refactoring (Last Resort)

Only when:
- Massive legacy codebase inherited
- Technical debt so large it blocks features
- Need dedicated time for architectural changes

**Danger:** Can become excuse to write messy code now, "fix later."

## Refactoring vs. Rewriting

| Refactoring | Rewriting |
|-------------|-----------|
| Incremental improvement | Start from scratch |
| Behavior unchanged | Often changes behavior |
| Tests pass throughout | No tests until done |
| Low risk | High risk |
| Ship while improving | Long development freeze |

### The Rewrite Trap

```pseudocode
// Tempting but dangerous
FUNCTION handle_legacy_system()
    // DON'T DO THIS:
    announce("We're rewriting everything!")
    spend_six_months_rewriting()
    discover_original_system_had_subtle_logic()
    spend_three_more_months_fixing()

    // DO THIS INSTEAD:
    WHILE system_not_ideal DO
        identify_worst_part()
        write_tests_for_that_part()
        refactor_incrementally()
        ship_improved_version()
    END WHILE
END FUNCTION
```

**When Rewrite is Justified:**
- Technology stack is obsolete and unmaintainable
- Cost of refactoring > cost of rewrite
- No tests exist and code is incomprehensible
- Business model has fundamentally changed

## Pseudocode Examples

### Example 1: Extract Method

**BEFORE:**
```pseudocode
FUNCTION process_order(order)
    // Calculate total
    total = 0
    FOR EACH item IN order.items DO
        total = total + item.price * item.quantity
    END FOR

    // Apply discount
    IF order.customer.is_premium THEN
        total = total * 0.9
    END IF

    // Add tax
    tax = total * 0.07
    total = total + tax

    // Send invoice
    invoice = create_invoice(order, total)
    email_service.send(order.customer.email, invoice)

    RETURN total
END FUNCTION
```

**AFTER:**
```pseudocode
FUNCTION process_order(order)
    total = calculate_total(order)
    send_invoice(order, total)
    RETURN total
END FUNCTION

FUNCTION calculate_total(order)
    subtotal = sum_items(order.items)
    discounted = apply_discount(subtotal, order.customer)
    RETURN add_tax(discounted)
END FUNCTION

FUNCTION sum_items(items)
    total = 0
    FOR EACH item IN items DO
        total = total + item.price * item.quantity
    END FOR
    RETURN total
END FUNCTION

FUNCTION apply_discount(amount, customer)
    IF customer.is_premium THEN
        RETURN amount * 0.9
    ELSE
        RETURN amount
    END IF
END FUNCTION

FUNCTION add_tax(amount)
    RETURN amount * 1.07
END FUNCTION
```

### Example 2: Replace Type Code with Polymorphism

**BEFORE:**
```pseudocode
CLASS Employee
    name
    type  // 1 = engineer, 2 = manager, 3 = salesperson

    FUNCTION calculate_pay()
        IF type == 1 THEN
            RETURN base_salary + overtime * rate
        ELSE IF type == 2 THEN
            RETURN base_salary + bonus
        ELSE IF type == 3 THEN
            RETURN base_salary + commission
        END IF
    END FUNCTION
END CLASS
```

**AFTER:**
```pseudocode
ABSTRACT CLASS Employee
    name
    base_salary

    ABSTRACT FUNCTION calculate_pay()
END CLASS

CLASS Engineer EXTENDS Employee
    overtime_hours
    hourly_rate

    FUNCTION calculate_pay()
        RETURN base_salary + (overtime_hours * hourly_rate)
    END FUNCTION
END CLASS

CLASS Manager EXTENDS Employee
    bonus

    FUNCTION calculate_pay()
        RETURN base_salary + bonus
    END FUNCTION
END CLASS

CLASS Salesperson EXTENDS Employee
    commission

    FUNCTION calculate_pay()
        RETURN base_salary + commission
    END FUNCTION
END CLASS
```

## Summary Table

| Aspect | Key Principle |
|--------|---------------|
| **Frequency** | Continuously, as part of daily work |
| **Safety Net** | Comprehensive automated tests |
| **Step Size** | Smallest change possible, then test |
| **Triggers** | Code smells, duplication, adding features |
| **Goal** | Improve structure without changing behavior |
| **Timing** | Real-time > scheduled refactoring |
| **Alternative** | Refactor incrementally >> rewrite from scratch |
| **Commitment** | Every commit leaves code better than found |

## Key Takeaways

1. **Refactor ruthlessly, but safely** - Always have tests
2. **Small steps win** - One thing at a time, run tests
3. **Real-time refactoring** - Don't wait for "refactoring sprint"
4. **Scout rule** - Leave code better than you found it
5. **Avoid rewrites** - Incremental improvement beats Big Bang
6. **Learn the catalog** - Know common refactorings by name
7. **Code smells are signals** - Listen to them

---

*Based on concepts from "The Pragmatic Programmer" by David Thomas and Andrew Hunt.*
