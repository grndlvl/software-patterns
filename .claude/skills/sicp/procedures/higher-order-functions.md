# Higher-Order Functions

## Definition

Higher-order functions are procedures that manipulate procedures by taking functions as arguments and/or returning functions as values. They abstract common computational patterns and enable composable, reusable code.

**Core characteristic:** A function is higher-order if it either:
1. Takes one or more functions as arguments
2. Returns a function as its result
3. Both

---

## Procedures as Arguments

### Summation Abstraction

The classic pattern: extract summation logic from specific implementations.

**Problem:** Computing different sums requires repetitive code:

```pseudocode
// Sum of integers 1 to n
sum_integers(n):
  result = 0
  i = 1
  while i <= n:
    result = result + i
    i = i + 1
  return result

// Sum of squares 1² to n²
sum_squares(n):
  result = 0
  i = 1
  while i <= n:
    result = result + (i * i)
    i = i + 1
  return result

// Sum of cubes 1³ to n³
sum_cubes(n):
  result = 0
  i = 1
  while i <= n:
    result = result + (i * i * i)
    i = i + 1
  return result
```

**Solution:** Abstract the pattern by passing the transformation function:

```pseudocode
sum(term, a, b):
  "Sum of term(k) for k from a to b"
  if a > b:
    return 0
  else:
    return term(a) + sum(term, a + 1, b)

// Usage:
identity(x) = x
square(x) = x * x
cube(x) = x * x * x

sum_integers(n) = sum(identity, 1, n)
sum_squares(n) = sum(square, 1, n)
sum_cubes(n) = sum(cube, 1, n)
```

**Benefits:**
- Eliminates code duplication
- Single implementation handles infinite variations
- Business logic (what to sum) separated from control flow (how to sum)

---

## Procedures as Return Values

### Derivative

A function that returns a function approximating the derivative:

```pseudocode
// Approximation: f'(x) ≈ (f(x+h) - f(x)) / h for small h
deriv(f, h):
  "Return a function that computes the approximate derivative of f"
  return function(x):
    return (f(x + h) - f(x)) / h

// Usage:
square(x) = x * x
approx_derivative_of_square = deriv(square, 0.00001)
approx_derivative_of_square(3)  // Returns ≈ 6.00003 (correct derivative is 6)
```

### Compose

Combine multiple functions into a single function:

```pseudocode
compose(f, g):
  "Return a new function h(x) = f(g(x))"
  return function(x):
    return f(g(x))

// Usage:
square(x) = x * x
add_one(x) = x + 1

square_then_add_one = compose(add_one, square)
square_then_add_one(3)  // Returns 10: (3² + 1 = 10)
```

### Partial Application / Currying

Transform a multi-argument function into nested single-argument functions:

```pseudocode
multiply(x, y) = x * y

curry_multiply(x):
  "Return a function that multiplies by x"
  return function(y):
    return multiply(x, y)

// Usage:
double = curry_multiply(2)
double(5)  // Returns 10

// Or directly:
triple = curry_multiply(3)
triple(7)  // Returns 21
```

---

## Lambda Expressions

Anonymous functions defined inline for use with higher-order functions:

```pseudocode
// Named function
square(x) = x * x

// Equivalent lambda expression
λ(x) -> x * x

// Using lambda with sum:
sum(λ(x) -> x * x, 1, 5)  // Sum of squares from 1 to 5

// Using lambda with compose:
compose(λ(x) -> x + 1, λ(x) -> x * x)  // Add one to the square

// Predicate for filtering:
filter(λ(x) -> x > 0, [-2, 3, -1, 4, 0])  // Returns [3, 4]
```

**Benefits:**
- Avoid naming trivial one-time-use functions
- Keep logic close to where it's used
- More concise code

---

## Common Patterns

### Map

Apply a function to each element of a sequence, returning a new sequence:

```pseudocode
map(procedure, sequence):
  "Apply procedure to each element of sequence"
  if sequence is empty:
    return []
  else:
    return [procedure(first(sequence))] + map(procedure, rest(sequence))

// Usage:
square(x) = x * x
map(square, [1, 2, 3, 4])  // Returns [1, 4, 9, 16]
map(λ(x) -> x + 1, [10, 20, 30])  // Returns [11, 21, 31]
```

**Time Complexity:** O(n)  
**Space Complexity:** O(n)

### Filter

Keep only elements satisfying a predicate:

```pseudocode
filter(predicate, sequence):
  "Keep elements where predicate(element) is true"
  if sequence is empty:
    return []
  else if predicate(first(sequence)):
    return [first(sequence)] + filter(predicate, rest(sequence))
  else:
    return filter(predicate, rest(sequence))

// Usage:
is_even(x) = (x mod 2) == 0
filter(is_even, [1, 2, 3, 4, 5, 6])  // Returns [2, 4, 6]

// With lambda:
filter(λ(x) -> x > 0, [-2, 3, -1, 4, 0])  // Returns [3, 4]
```

**Time Complexity:** O(n)  
**Space Complexity:** O(n) worst case

### Fold / Reduce

Accumulate a sequence into a single value:

```pseudocode
fold(combiner, initial_value, sequence):
  "Combine elements using combiner, starting with initial_value"
  if sequence is empty:
    return initial_value
  else:
    return fold(combiner, 
                combiner(initial_value, first(sequence)), 
                rest(sequence))

// Usage:
add(x, y) = x + y
fold(add, 0, [1, 2, 3, 4])  // Returns 10 (0+1+2+3+4)

multiply(x, y) = x * y
fold(multiply, 1, [1, 2, 3, 4])  // Returns 24 (factorial-like: 1*1*2*3*4)

// With lambda:
fold(λ(acc, x) -> acc + (x * x), 0, [1, 2, 3])  // Returns 14 (1² + 2² + 3²)
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1) if tail-recursive, O(n) if naive recursion

### Compose

Chain multiple functions:

```pseudocode
compose(f, g):
  "Return h where h(x) = f(g(x))"
  return λ(x) -> f(g(x))

compose_many(f1, f2, f3, ..., fn):
  "Compose all functions: f1(f2(f3(...fn(x))))"
  if only one function:
    return that function
  else:
    return compose(first_function, compose_many(rest of functions))

// Usage:
add_one(x) = x + 1
double(x) = x * 2
square(x) = x * x

h = compose(add_one, compose(square, double))
h(3)  // Returns 37: ((3 * 2)² + 1 = (6)² + 1 = 37)

// Multi-compose:
transform = compose_many(add_one, square, double)
transform(5)  // Returns 101: (((5 * 2)² + 1) = ((10)² + 1) = 101)
```

**Time Complexity:** O(n) in number of functions, O(1) per function application  
**Space Complexity:** O(n) for composition chain

---

## Currying and Partial Application

### Currying

Transform multi-argument functions into chains of single-argument functions:

```pseudocode
// Original function
add(x, y) = x + y

// Curried version
curry_add(x):
  return λ(y) -> x + y

// Usage:
add_five = curry_add(5)
add_five(3)  // Returns 8

// Multi-argument currying:
curry_multiply(x):
  return λ(y):
    return λ(z) -> x * y * z

multiply_three = curry_multiply(2)
multiply_six = multiply_three(3)
multiply_six(4)  // Returns 24 (2 * 3 * 4)
```

### Partial Application

Fix some arguments while leaving others variable:

```pseudocode
partial(fn, fixed_args):
  "Return a function with some arguments pre-filled"
  return λ(remaining_args) -> fn(fixed_args + remaining_args)

// Usage with multiply:
multiply(x, y) = x * y

double = partial(multiply, [2])
double(5)  // Returns 10

// With more complex function:
string_concat(prefix, text, suffix) = prefix + text + suffix

greeting = partial(string_concat, ["Hello, "])
greet_user = partial(greeting, [", welcome!"])
greet_user("Alice")  // Returns "Hello, Alice, welcome!"
```

### Difference

| Aspect | Currying | Partial Application |
|--------|----------|---------------------|
| **Transforms** | All arguments → single-arg functions | Some arguments → new function |
| **Structure** | Nested single-arg functions | Returns function with reduced arity |
| **Use Case** | Function composition chains | Reusing functions with fixed context |
| **Example** | `curry(f)(a)(b)(c)` | `partial(f, a)(b, c)` |

---

## Benefits

| Benefit | How It Helps |
|---------|-------------|
| **Code Reusability** | Abstract patterns apply to infinite variations (e.g., sum works for any term function) |
| **Separation of Concerns** | Business logic (what) separated from control flow (how) |
| **Composability** | Combine simple functions into complex operations |
| **Testability** | Small, focused functions easier to test |
| **Readability** | Intent explicit via function names and lambdas |
| **DRY Principle** | Eliminate repetitive code patterns |
| **Abstraction Levels** | Build abstractions on top of abstractions |
| **Lazy Evaluation** | Functions returned but not executed until needed |
| **Functional Purity** | Higher-order functions encourage side-effect-free code |

---

## Summary Table

| Pattern | Purpose | Common Operations |
|---------|---------|-------------------|
| **Procedures as Arguments** | Abstract repetitive patterns | sum, filter, search, accumulate |
| **Procedures as Return Values** | Create specialized functions | derivative, compose, partial application |
| **Map** | Transform each element | square all, increment all, format all |
| **Filter** | Select matching elements | find evens, find positives, find valid items |
| **Fold/Reduce** | Combine into single value | sum, product, max, count, concatenate |
| **Compose** | Chain operations | build pipelines, create complex transforms |
| **Currying** | Enable partial application | create specialized versions, build chains |
| **Lambda Expressions** | Inline anonymous functions | avoid naming trivial functions |

---

## Key Principles

1. **Functions are First-Class Values**
   - Can be stored in variables
   - Can be passed as arguments
   - Can be returned from functions
   - Can be combined and composed

2. **Abstraction Through Parametrization**
   - Extract common patterns
   - Pass variation as functions
   - One implementation handles infinite variations

3. **Composition Over Inheritance**
   - Build complex behavior by combining simple functions
   - More flexible than class hierarchies
   - Easier to reason about

4. **Separation of Concerns**
   - What to do (business logic) vs. how to do it (control flow)
   - What to transform (values) vs. how to transform (function)

---

*Based on concepts from "Structure and Interpretation of Computer Programs" by Abelson and Sussman.*
