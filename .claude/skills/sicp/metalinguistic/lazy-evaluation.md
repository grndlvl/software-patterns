# Lazy Evaluation

Lazy evaluation (or normal-order evaluation) defers computation until results are actually needed. This powerful technique enables infinite data structures, improved efficiency, and elegantly expresses certain algorithms.

## Table of Contents

1. [Evaluation Order Fundamentals](#evaluation-order-fundamentals)
2. [Thunks: The Foundation](#thunks-the-foundation)
3. [Lazy Evaluation in Interpreters](#lazy-evaluation-in-interpreters)
4. [Lazy Data Structures](#lazy-data-structures)
5. [Benefits](#benefits)
6. [Drawbacks](#drawbacks)
7. [Comparison Tables](#comparison-tables)

---

## Evaluation Order Fundamentals

### Applicative Order (Strict Evaluation)

Arguments are **evaluated before** function application.

```pseudocode
function sum_of_squares(x, y)
    return square(x) + square(y)

function square(n)
    return n * n

// Applicative order evaluation
sum_of_squares(3, 4)
→ sum_of_squares(3, 4)           // args already values
→ square(3) + square(4)
→ (3 * 3) + (4 * 4)
→ 9 + 16
→ 25
```

**Characteristics:**
- Predictable execution order
- Function sees fully evaluated arguments
- Efficiently avoids redundant computation (in strict languages)

### Normal Order (Lazy Evaluation)

Arguments are **substituted unevaluated** until needed.

```pseudocode
// Normal order evaluation
sum_of_squares(3, 4)
→ square(3) + square(4)          // arguments substituted, NOT evaluated
→ (3 * 3) + (4 * 4)              // only evaluated when needed
→ 9 + 16
→ 25
```

**Characteristics:**
- Arguments evaluated only if actually used
- Can handle infinite sequences
- May evaluate arguments multiple times (without memoization)

### Key Difference: Conditional Example

```pseudocode
function if_greater(pred, then_val, else_val)
    if pred
        return then_val
    else
        return else_val

// Applicative order: both branches evaluated
if_greater(true, expensive_calc_1(), expensive_calc_2())
→ both expensive_calc_1() and expensive_calc_2() execute!

// Normal order: only needed branch evaluated
if_greater(true, expensive_calc_1(), expensive_calc_2())
→ only expensive_calc_1() executes
```

---

## Thunks: The Foundation

A **thunk** is a closure that packages an expression for delayed evaluation.

### Creating Thunks

```pseudocode
// Thunk: closure capturing expression + environment
procedure delay(expr, env)
    return lambda() eval(expr, env)

// Example
thunk_x = delay(2 + 3, current_env)
// thunk_x is now a lambda that will compute 2 + 3 when called

// Force evaluation
result = thunk_x()  // → 5
```

### Representing Thunks

```pseudocode
// Option 1: As lambda
make_thunk(expr) = lambda() eval(expr)
force_thunk(thunk) = thunk()

// Option 2: As tagged data structure
make_thunk(expr) = list('thunk', expr)
force_thunk(thunk) = eval(thunk[1])
is_thunk(x) = is_list(x) and x[0] == 'thunk'
```

### Simple Thunk Example

```pseudocode
// Create thunks without explicit force/delay
x_thunk = lambda() 10
y_thunk = lambda() 20

// Force when needed
x = x_thunk()    // → 10
y = y_thunk()    // → 20
sum = x + y      // → 30

// Thunks delay side effects
counter = 0

increment_thunk = lambda()
    counter = counter + 1
    return counter

// Counter still 0
print(counter)        // → 0

// After forcing thunk
result = increment_thunk()
print(counter)        // → 1
print(result)         // → 1
```

---

## Forcing Thunks

### Explicit Force

```pseudocode
procedure force(thunk)
    if is_thunk(thunk)
        return thunk()
    else
        return thunk

// Usage
value = force(delayed_expr)
```

### Implicit Force in Evaluation

```pseudocode
// Lazy evaluator automatically forces when needed
procedure eval(expr, env)
    case expr of
        number? → expr
        variable? → lookup(expr, env)
        quote? → eval_quoted(expr)
        lambda? → make_procedure(expr, env)
        
        procedure_call? →
            func = eval(expr.function, env)
            // DON'T eval args - create thunks instead
            args = map(lambda(arg) delay(arg, env), 
                       expr.arguments)
            apply(func, args)

procedure apply(func, args)
    body = func.body
    params = func.parameters
    env = extend_env(params, args, func.env)
    eval(body, env)
    
// When variable accessed in body, automatically forced
procedure lookup(var, env)
    val = env_lookup(var, env)
    if is_thunk(val)
        return force(val)  // Auto-force
    else
        return val
```

---

## Memoization of Thunks

Without memoization, thunks are re-evaluated every time forced.

### Problem: Redundant Computation

```pseudocode
expensive_thunk = lambda() calculate_prime_factors(10000000)

// Without memoization
result1 = force(expensive_thunk)  // compute
result2 = force(expensive_thunk)  // compute AGAIN
result3 = force(expensive_thunk)  // compute AGAIN
```

### Solution: Memoized Thunk

```pseudocode
procedure make_memoized_thunk(expr, env)
    evaluated = false
    result = null
    
    return lambda()
        if not evaluated
            result = eval(expr, env)
            evaluated = true
        return result

// Usage
memo_thunk = make_memoized_thunk(expensive_calc(), env)

force(memo_thunk)  // compute
force(memo_thunk)  // return cached result
force(memo_thunk)  // return cached result
```

### Automatic Memoization in Lazy Evaluator

```pseudocode
procedure eval(expr, env)
    // ... existing cases ...
    
    procedure_call? →
        func = eval(expr.function, env)
        args = map(
            lambda(arg) make_memoized_thunk(arg, env),
            expr.arguments
        )
        apply(func, args)
```

### Memoization Trade-offs

| Aspect | Memoized | Non-Memoized |
|--------|----------|--------------|
| **Performance** | Faster after first force | May recompute |
| **Memory** | Stores cached value | Can GC intermediate result |
| **Side Effects** | Executes once | May execute multiple times |
| **Correctness** | Ensures consistency | Risky with state |

---

## Lazy Evaluation in Interpreters

### Lazy Evaluator Structure

```pseudocode
// Main evaluation loop
procedure eval(expr, env)
    case expr of
        self_evaluating? → expr
        
        variable? → 
            val = env_lookup(expr, env)
            force(val)
        
        quote? → expr.quoted_expr
        
        if? →
            pred = eval(expr.predicate, env)
            if pred
                eval(expr.consequent, env)
            else
                eval(expr.alternative, env)
        
        lambda? → make_procedure(expr, env)
        
        procedure_call? →
            func = eval(expr.function, env)
            args = map(
                lambda(arg) delay(arg, env),
                expr.arguments
            )
            apply(func, args)

procedure apply(proc, args)
    case proc of
        primitive? → 
            // Primitives must force arguments
            forced_args = map(force, args)
            return proc.function(forced_args)
        
        compound? →
            new_env = extend_env(
                proc.parameters,
                args,
                proc.env
            )
            eval(proc.body, new_env)
```

### Special Forms in Lazy Evaluator

Some forms must be special-cased to avoid forcing:

```pseudocode
// IF: conditional branch - don't force all branches
if_special(expr, env)
    pred = force(eval(expr.predicate, env))
    if pred
        return eval(expr.consequent, env)
    else
        return eval(expr.alternative, env)

// AND: short-circuit - don't force remaining
and_special(exprs, env)
    if empty?(exprs)
        return true
    if not force(eval(first(exprs), env))
        return false
    return and_special(rest(exprs), env)

// OR: short-circuit
or_special(exprs, env)
    if empty?(exprs)
        return false
    if force(eval(first(exprs), env))
        return true
    return or_special(rest(exprs), env)
```

---

## Lazy Data Structures

### Lazy Pairs

```pseudocode
// Standard pair (eager)
cons_eager(head, tail) = list(head, tail)

// Lazy pair - delay tail
cons_lazy(head, tail_thunk) = list(head, tail_thunk)

car(pair) = pair[0]        // head always available

cdr(pair) = force(pair[1]) // force the tail thunk

// Example
pair = cons_lazy(1, lambda() cons_lazy(2, lambda() cons_lazy(3, nil)))

car(pair)           // → 1
car(cdr(pair))      // → 2
car(cdr(cdr(pair))) // → 3
```

### Lazy Lists (Streams)

```pseudocode
// Stream: pair with lazy tail
stream_cons(head, tail_stream) = cons_lazy(head, tail_stream)

stream_car(stream) = car(stream)

stream_cdr(stream) = cdr(stream)

stream_null?(stream) = stream == nil

// Example: infinite stream of integers
procedure integers_from(n)
    return stream_cons(
        n,
        lambda() integers_from(n + 1)
    )

integers = integers_from(1)

stream_car(integers)                    // → 1
stream_car(stream_cdr(integers))        // → 2
stream_car(stream_cdr(stream_cdr(integers)))  // → 3
// Never computed beyond what we access
```

### Lazy List Operations

```pseudocode
procedure stream_map(f, stream)
    if stream_null?(stream)
        return nil
    return stream_cons(
        f(stream_car(stream)),
        lambda() stream_map(f, stream_cdr(stream))
    )

procedure stream_filter(predicate, stream)
    if stream_null?(stream)
        return nil
    head = stream_car(stream)
    if predicate(head)
        return stream_cons(
            head,
            lambda() stream_filter(predicate, stream_cdr(stream))
        )
    else
        return stream_filter(predicate, stream_cdr(stream))

procedure stream_take(n, stream)
    if n <= 0 or stream_null?(stream)
        return nil
    return cons(
        stream_car(stream),
        stream_take(n - 1, stream_cdr(stream))
    )

// Usage: infinite stream of integers
ints = integers_from(1)

// Filter to get odd numbers
odds = stream_filter(lambda(x) x mod 2 == 1, ints)

// Take first 5 odds
first_5_odds = stream_take(5, odds)
// → (1 3 5 7 9)
```

### Infinite Stream Examples

```pseudocode
// Infinite stream of ones
procedure ones()
    return stream_cons(1, lambda() ones())

// Fibonacci stream
procedure fibs()
    procedure fib_gen(a, b)
        return stream_cons(a, lambda() fib_gen(b, a + b))
    return fib_gen(0, 1)

fibs_stream = fibs()
stream_take(10, fibs_stream)
// → (0 1 1 2 3 5 8 13 21 34)
// Only computed the first 10 values

// Prime sieve (Sieve of Eratosthenes)
procedure primes()
    procedure sieve(stream)
        return stream_cons(
            stream_car(stream),
            lambda()
                sieve(stream_filter(
                    lambda(x) x mod stream_car(stream) != 0,
                    stream_cdr(stream)
                ))
        )
    return sieve(integers_from(2))

primes_stream = primes()
stream_take(10, primes_stream)
// → (2 3 5 7 11 13 17 19 23 29)
```

---

## Benefits

### 1. Infinite Data Structures

```pseudocode
// Can represent infinite sequences as finite code
fibonacci_stream = fibs()

// Compute any fibonacci number
nth_fib = lambda(n) stream_car(stream_cdr_n(fibonacci_stream, n))

nth_fib(100)  // Works! Only computes what's needed
```

### 2. Computational Efficiency

```pseudocode
// Only compute branches taken
result = if condition
    then expensive_computation_1()
    else expensive_computation_2()
// Only one computation runs

// Short-circuit evaluation
result = a and b and c and d
// Stops at first false value
```

### 3. Elegant Modular Expression

```pseudocode
// Separation of concern: logic vs. execution
// Define stream transformations clearly
process = stream_filter(pred, stream_map(f, data))

// Without lazy evaluation, requires explicit iteration
// With lazy evaluation, declarative and composable
```

### 4. Resource Management

```pseudocode
// Don't need entire dataset in memory
large_file_lines = stream_from_file("huge_file.txt")

// Process line-by-line
processed = stream_map(parse_line, large_file_lines)

// Take first 1000
result = stream_take(1000, processed)
// Only 1000 lines in memory, not entire file
```

### 5. Avoids Unnecessary Computation

```pseudocode
// Without lazy evaluation
all_squares = map(square, range(1, 1000000))
first_ten = take(10, all_squares)
// Squares all 1,000,000 numbers!

// With lazy evaluation
all_squares = stream_map(square, integers_from(1))
first_ten = stream_take(10, all_squares)
// Only squares first 10 numbers
```

---

## Drawbacks

### 1. Debugging Difficulty

```pseudocode
// Execution order unclear
result = a + b + c + d

// When are computations performed?
// Hard to trace control flow
// Thunks delay errors until force time
```

### 2. Space Leaks

```pseudocode
// Unevaluated thunk accumulates in memory
procedure sum_to_n(n)
    if n == 0
        return 0
    return n + sum_to_n(n - 1)

// With lazy evaluation, thunk chain builds up
// (+ n (+ (n-1) (+ (n-2) ... )))
// Before final evaluation, entire chain in memory
// Should accumulate in tail-call fashion
```

### 3. Unpredictable Performance

```pseudocode
// Performance characteristics hidden
operation = compute_something()

// When does it actually run?
// First access? Multiple times?
// Depends on memoization, usage patterns

// Hard to optimize without understanding internals
```

### 4. Garbage Collection Pressure

```pseudocode
// Thunks keep references alive
procedure long_computation()
    x = expensive_calc_1()
    y = expensive_calc_2()
    
    // If x is thunk and not forced/released,
    // keeps resources alive until y computed
    return y

// Memory usage profile more complex
```

### 5. Interacts Poorly with Side Effects

```pseudocode
// When does I/O happen?
counter = 0

increment = lambda()
    counter = counter + 1
    print(counter)

process = increment()

// Was anything printed? When?
// With lazy evaluation, unclear

// Side-effectful code should generally be eager
```

---

## Comparison Tables

### Evaluation Order Comparison

| Aspect | Applicative Order | Normal Order |
|--------|-------------------|--------------|
| **Evaluation** | Arguments before function | Arguments substituted unevaluated |
| **Thunks** | Not used | Required for delays |
| **Memoization** | N/A | Prevents redundant computation |
| **Infinite Structures** | Impossible | Natural |
| **Short-Circuit Ops** | Can't implement natively | Native support |
| **Side Effects** | Predictable order | Unpredictable order |
| **Performance** | Eager cost | Lazy + memoization cost |
| **Memory** | Fixed (args evaluated once) | Thunk overhead + caching |
| **Debugging** | Clear execution order | Deferred/hidden execution |

### Thunk Operations

| Operation | Purpose | Example |
|-----------|---------|---------|
| **delay** | Create thunk from expression | `delay(2 + 3)` |
| **force** | Evaluate thunk | `force(thunk)` → 5 |
| **memoize** | Cache thunk result after first force | First force computes, rest use cache |
| **auto-force** | Implicitly force on variable access | Lazy evaluator feature |

### Lazy vs. Eager Stream Processing

| Characteristic | Eager List | Lazy Stream |
|---|---|---|
| **Memory Usage** | All elements in memory | Only accessed elements + thunks |
| **Infinite Support** | Impossible | Natural |
| **Composition** | Requires intermediate lists | Seamless chaining |
| **Evaluation Order** | All computed upfront | On-demand |
| **Reusability** | Copy entire list | Thunk can generate many times |
| **Example Size** | `[1, 2, 3, ..., 1000]` | `(1 → (2 → (3 → ...)))` |

### When to Use Each

| Scenario | Applicative Order | Normal Order |
|----------|-------------------|--------------|
| Small, finite data | ✓ Preferred | Unnecessary overhead |
| Expensive computation in unused branches | ✗ Wasteful | ✓ Efficient |
| Infinite sequences | ✗ Impossible | ✓ Perfect |
| I/O and side effects | ✓ Predictable | ✗ Unclear timing |
| Mathematical algorithms | ✓ Natural | ✓ Sometimes cleaner |
| Interactive programs | ✓ Expected | ✗ Confusing |
| Stream processing | ✓ Memory heavy | ✓ Memory efficient |
| Conditional operations | ✗ Evaluates both branches | ✓ Short-circuits |

---

## Summary

### Key Concepts

| Concept | Definition |
|---------|-----------|
| **Normal Order** | Substitute arguments unevaluated; evaluate only when needed |
| **Applicative Order** | Evaluate arguments before function application |
| **Thunk** | Closure capturing an unevaluated expression |
| **Delay** | Create thunk without forcing evaluation |
| **Force** | Evaluate thunk to get result |
| **Memoization** | Cache thunk result to avoid recomputation |
| **Lazy Pair** | Pair with thunked tail for delayed cdr |
| **Stream** | Infinite lazy sequence |
| **Short-Circuit** | Evaluate only necessary conditions |

### When Lazy Evaluation Shines

1. **Infinite structures** - Define without termination condition
2. **Large data** - Process sequentially without loading all
3. **Expensive branches** - Skip unused computations
4. **Modular code** - Separate logic from execution
5. **Interactive systems** - Demand-driven computation

### Implementation Checklist

- [ ] Understand normal vs. applicative order trade-offs
- [ ] Implement thunk creation (delay)
- [ ] Implement thunk evaluation (force)
- [ ] Add memoization to prevent redundant computation
- [ ] Modify evaluator to thunk arguments in function calls
- [ ] Auto-force variables on access
- [ ] Implement special forms (if, and, or) as non-strict
- [ ] Build lazy pair and stream abstractions
- [ ] Test with infinite sequences
- [ ] Profile for space leaks

---

*Based on concepts from "Structure and Interpretation of Computer Programs" by Abelson and Sussman.*
