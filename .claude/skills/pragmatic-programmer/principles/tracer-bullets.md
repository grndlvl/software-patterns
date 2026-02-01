# Tracer Bullets

## Definition

> "Tracer bullets let you home in on your target by trying things and seeing how close they land."
> — Dave Thomas & Andy Hunt

In warfare, tracer bullets glow so gunners can see where their shots are going and adjust. In software, tracer bullet development means building a thin end-to-end implementation to get immediate feedback.

## The Concept

Instead of building all components in isolation and integrating later, tracer bullet development:

1. Builds a minimal skeleton through all layers
2. Gets it working end-to-end immediately
3. Adds features incrementally
4. Provides continuous feedback

## Tracer Bullets vs. Traditional

### Traditional Approach

```
Phase 1: Build complete UI          [████████████░░░]
Phase 2: Build complete logic       [████████████░░░]
Phase 3: Build complete database    [████████████░░░]
Phase 4: Integrate (pray)           [░░░░░░░░░░░░░░░] ← Big bang!

Risk: Integration problems discovered late
```

### Tracer Bullet Approach

```
Sprint 1: Thin slice through all layers [█░░░░░░░░░░░░░░]
Sprint 2: Add more features             [███░░░░░░░░░░░░]
Sprint 3: Add more features             [█████░░░░░░░░░░]
Sprint 4: Add more features             [███████░░░░░░░░]

Benefit: Working system from day one
```

## Example: Building a Report System

### Traditional (Don't Do This)

```
Month 1: Design all report types, all formats
Month 2: Build report engine (complete)
Month 3: Build all data queries
Month 4: Build all UI templates
Month 5: Integrate and debug
Month 6: Still debugging integration issues...
```

### Tracer Bullet (Do This)

```
Week 1:
- ONE report type (Sales Summary)
- ONE output format (HTML)
- ONE data query (hardcoded SQL)
- ONE simple UI
- Working end-to-end!

Week 2:
- Add PDF output
- Still just Sales Summary
- Working end-to-end!

Week 3:
- Add second report type (Inventory)
- Working end-to-end!

Week 4:
- Add parameterized queries
- Working end-to-end!
```

## Tracer Bullet Code

```pseudocode
// Week 1: Minimal tracer bullet

// Presentation: Simple, hardcoded
class ReportController {
    function showReport() {
        data = reportService.getSalesSummary()
        return htmlTemplate.render(data)
    }
}

// Business Logic: Minimal
class ReportService {
    function getSalesSummary() {
        return repository.querySales()
    }
}

// Data: Hardcoded query
class ReportRepository {
    function querySales() {
        return database.query("SELECT * FROM sales WHERE date > '2024-01-01'")
    }
}

// It's not complete, but it WORKS end-to-end!
// We can demo it, get feedback, and iterate.
```

## Benefits

### 1. Users See Something Working

```
Day 1: "Here's a basic report. Is this the right direction?"
User: "Yes, but can we add totals?"
Day 2: "Here's totals. What about grouping?"
User: "Perfect! But we need it in PDF too."
Day 3: "Here's PDF export..."
```

### 2. Developers Have a Structure

New code plugs into existing skeleton:
- New report types extend existing patterns
- New formats fit existing architecture
- Team members can work in parallel

### 3. Integration is Continuous

No "integration phase" because you're always integrated.

### 4. Progress is Visible

```
✓ Basic report works
✓ PDF export works
✓ Date filtering works
□ Multiple report types
□ Scheduled generation
□ Email delivery
```

### 5. You Have Something to Demo

At any point, you have a working (if incomplete) system to show stakeholders.

## Tracer Bullets vs. Prototypes

| Tracer Bullets | Prototypes |
|----------------|------------|
| Code you keep | Code you throw away |
| Production quality | Quick and dirty |
| Lean but complete | May skip layers |
| Evolves into final system | Used to learn, then discarded |
| Real architecture | May be architectural spike |

```pseudocode
// Prototype: Quick, throwaway
function canWeDoThis() {
    // Hacky code to answer a question
    // Will be rewritten properly
}

// Tracer Bullet: Minimal but production-ready
function createOrder(items) {
    // Simple but correct
    // Will be extended, not rewritten
    order = new Order(items)
    repository.save(order)
    return order
}
```

## When to Use Tracer Bullets

✅ **Use when:**
- Building new system with uncertain requirements
- Team needs to see progress
- You're unsure how pieces will fit together
- Stakeholders need early feedback
- You want to reduce integration risk

❌ **Don't use when:**
- Requirements are crystal clear and fixed
- You're adding to a well-established system
- The system is trivial

## Implementing Tracer Bullets

### Step 1: Identify the Layers

```
UI → API → Service → Repository → Database
```

### Step 2: Pick ONE Feature

Choose a feature that touches all layers:
- User registration (UI form → API → save to DB → return)
- View single item (DB → API → UI display)
- Simple search (UI input → API → query → results)

### Step 3: Build Thin Slice

```pseudocode
// Minimal UI
function RegistrationForm() {
    return <form onSubmit={api.register}>
        <input name="email" />
        <button>Register</button>
    </form>
}

// Minimal API
function register(request) {
    email = request.body.email
    user = userService.register(email)
    return { id: user.id }
}

// Minimal Service
function register(email) {
    user = new User(email)
    return repository.save(user)
}

// Minimal Repository
function save(user) {
    database.insert("users", user)
    return user
}
```

### Step 4: Verify End-to-End

Actually run it:
1. Fill out form
2. Click submit
3. Check database
4. See confirmation

### Step 5: Iterate

Add features one at a time, always keeping it working.

## Summary

> "Look for the important requirements, the ones that define the system. Look for the areas where you have doubts, and where you see the biggest risks. Then prioritize your development so that these are the first areas you code."

Tracer bullets give you:
- **Immediate feedback** - Know if you're on target
- **Continuous integration** - Always working together
- **Visible progress** - Something to demo
- **A framework for growth** - Structure to build on

---

*Based on concepts from "The Pragmatic Programmer" by David Thomas and Andrew Hunt.*
