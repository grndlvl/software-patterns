# Streams: Delayed Sequences and Lazy Evaluation

## Definition

A **stream** is a data structure representing a sequence of values where elements are computed on-demand rather than eagerly pre-computed. Streams enable:
- **Lazy evaluation**: Elements are not calculated until explicitly requested
- **Delayed sequences**: Deferred computation of list operations
- **Infinite sequences**: Represent unbounded collections without infinite memory
- **Modular composition**: Separate sequence generation from sequence processing

Streams decouple the order of computations from the order of evaluations, allowing programs to manipulate sequences without materializing all elements in memory.

---

## Core Stream Primitives

### Fundamental Operations

```
cons-stream(head, tail)
  Returns a stream with the given head element and delayed tail
  tail is NOT evaluated immediately; it is wrapped in a thunk

stream-car(stream)
  Returns the first element of the stream (the head)
  No forced evaluation needed

stream-cdr(stream)
  Returns the rest of the stream (the tail)
  Forces the evaluation of the delayed tail
  Returns a stream, not a simple value

stream-null?(stream)
  Predicate: true if stream is empty, false otherwise

the-empty-stream
  Constant representing an empty stream
  Analogous to null in list processing
```

### Example Stream Construction

```
stream = cons-stream(1, cons-stream(2, cons-stream(3, the-empty-stream)))

stream-car(stream)
  → 1

stream-car(stream-cdr(stream))
  → 2

stream-car(stream-cdr(stream-cdr(stream)))
  → 3
```

---

## Delay and Force Primitives

**Delay** and **force** are the mechanisms enabling lazy evaluation in streams.

### Delay: Creating Thunks

```
delay(expression)
  Wraps expression in a thunk (unevaluated computation)
  Does NOT execute expression immediately
  Returns a thunk object containing the expression
  
Example:
  delayed_sum = delay(3 + 4)
  -- At this point, 3 + 4 has not been computed
```

### Force: Evaluating Thunks

```
force(thunk)
  Evaluates a delayed expression (thunk)
  Returns the result of executing the wrapped expression
  
Example:
  delayed_sum = delay(3 + 4)
  force(delayed_sum)
    → 7
```

### Memoization Pattern

```
-- Thunks should be memoized to avoid recomputation
-- After first force, cache the result

create_memoized_thunk(expression)
  already_computed = false
  cached_value = null
  
  thunk = lambda:
    if not already_computed
      cached_value = evaluate(expression)
      already_computed = true
    return cached_value
  
  return thunk

force(thunk)
  return thunk()  -- Calls the thunk, returns cached value on subsequent calls
```

---

## Stream Operations

### Basic Mapping and Filtering

```
stream-map(f, stream)
  if stream-null?(stream)
    return the-empty-stream
  else
    return cons-stream(
      f(stream-car(stream)),
      delay(stream-map(f, stream-cdr(stream)))
    )

Example:
  integers = [1, 2, 3, 4, 5, ...]
  stream-map(lambda x: x * x, integers)
    → [1, 4, 9, 16, 25, ...]
```

### Stream Filtering

```
stream-filter(predicate, stream)
  if stream-null?(stream)
    return the-empty-stream
  else if predicate(stream-car(stream))
    return cons-stream(
      stream-car(stream),
      delay(stream-filter(predicate, stream-cdr(stream)))
    )
  else
    return stream-filter(predicate, stream-cdr(stream))

Example:
  integers = [1, 2, 3, 4, 5, 6, 7, 8, 9, ...]
  stream-filter(lambda x: x mod 2 = 0, integers)
    → [2, 4, 6, 8, 10, ...]
```

### Stream Reference (nth element)

```
stream-ref(stream, n)
  if n = 0
    return stream-car(stream)
  else
    return stream-ref(stream-cdr(stream), n - 1)

Example:
  fibonacci = [0, 1, 1, 2, 3, 5, 8, 13, ...]
  stream-ref(fibonacci, 5)
    → 5
```

### Taking First N Elements

```
stream-take(stream, n)
  if n = 0 or stream-null?(stream)
    return the-empty-stream
  else
    return cons-stream(
      stream-car(stream),
      delay(stream-take(stream-cdr(stream), n - 1))
    )

Example:
  fibonacci = [0, 1, 1, 2, 3, 5, 8, 13, ...]
  stream-take(fibonacci, 5)
    → [0, 1, 1, 2, 3]
```

### Stream Concatenation

```
stream-append(stream1, stream2)
  if stream-null?(stream1)
    return stream2
  else
    return cons-stream(
      stream-car(stream1),
      delay(stream-append(stream-cdr(stream1), stream2))
    )
```

---

## Infinite Streams

### Integers Stream

```
integers-starting-from(n)
  return cons-stream(
    n,
    delay(integers-starting-from(n + 1))
  )

integers = integers-starting-from(1)
  → [1, 2, 3, 4, 5, 6, 7, ...]
  
-- No infinite loop! Elements computed only when accessed
stream-ref(integers, 100)
  → 101
```

### Fibonacci Stream

```
fibonacci-stream()
  return fib-iter(0, 1)

fib-iter(a, b)
  return cons-stream(
    a,
    delay(fib-iter(b, a + b))
  )

fib = fibonacci-stream()
  → [0, 1, 1, 2, 3, 5, 8, 13, 21, 34, ...]
  
-- Generates Fibonacci numbers lazily
-- No pre-computation of entire sequence
```

### Even Numbers Stream (via Filter)

```
even-integers()
  return stream-filter(
    lambda x: x mod 2 = 0,
    integers-starting-from(1)
  )

evens = even-integers()
  → [2, 4, 6, 8, 10, 12, ...]
  
-- Computed on demand, not pre-filtered
```

### Sieve of Eratosthenes (Primes)

```
sieve(stream)
  return cons-stream(
    stream-car(stream),
    delay(sieve(
      stream-filter(
        lambda x: x mod stream-car(stream) != 0,
        stream-cdr(stream)
      )
    ))
  )

primes = sieve(integers-starting-from(2))
  → [2, 3, 5, 7, 11, 13, 17, 19, 23, 29, ...]
  
-- Generates infinite prime sequence
-- Each prime computed only when needed
```

### Stream of Pairs (Cartesian Product)

```
pairs(stream1, stream2)
  return cons-stream(
    [stream-car(stream1), stream-car(stream2)],
    delay(interleave(
      stream-map(
        lambda x: [stream-car(stream1), x],
        stream-cdr(stream2)
      ),
      pairs(stream-cdr(stream1), stream2)
    ))
  )

interleave(stream1, stream2)
  if stream-null?(stream1)
    return stream2
  else
    return cons-stream(
      stream-car(stream1),
      delay(interleave(stream2, stream-cdr(stream1)))
    )

-- Generates all pairs of positive integers
p = pairs(integers-starting-from(1), integers-starting-from(1))
  → [[1,1], [1,2], [2,1], [1,3], [2,2], [3,1], [1,4], ...]
```

---

## Stream Processing vs List Processing

### List Processing (Eager Evaluation)

```
-- Pre-compute all elements
list = [1, 2, 3, 4, 5]

-- Each operation creates intermediate lists in memory
result = filter(lambda x: x > 2, list)
  → [3, 4, 5]  -- Full list created

mapped = map(lambda x: x * x, result)
  → [9, 16, 25]  -- Another full list created

-- All elements computed even if not all used
```

### Stream Processing (Lazy Evaluation)

```
-- Define infinite sequences without pre-computation
stream = integers-starting-from(1)
  -- Represented as a computation, not a data structure

-- Operations are composed without materializing intermediates
result = stream-map(
  lambda x: x * x,
  stream-filter(lambda x: x > 2, stream)
)

-- Access only what's needed
stream-ref(result, 2)
  → 16  -- Only computes [1→3, 4→16] on demand
  
-- Never materializes full filtered or mapped streams
```

### Memory Efficiency Example

```
-- List: Pre-compute 1 million numbers, filter, square
numbers = range(1, 1000000)         -- 1M elements in memory
large_list = filter(is_even, numbers)  -- Still 500K elements
squared = map(square, large_list)      -- Still 500K elements

-- Stream: Compute only elements accessed
numbers = integers-starting-from(1)
filtered = stream-filter(is_even, numbers)
squared = stream-map(square, filtered)
stream-ref(squared, 100)  -- Computes only elements 1-202 on demand
```

---

## Implicit Definition of Streams

Streams enable **mutually recursive definitions** where streams are defined in terms of themselves.

### Fibonacci via Self-Reference

```
-- Classic explicit definition (eager)
fib(n) = if n <= 1 then n else fib(n-1) + fib(n-2)

-- Stream implicit definition (lazy)
fibs = cons-stream(0, cons-stream(1, 
  stream-add(fibs, stream-cdr(fibs))
))

stream-add(stream1, stream2)
  if stream-null?(stream1)
    return the-empty-stream
  else
    return cons-stream(
      stream-car(stream1) + stream-car(stream2),
      delay(stream-add(stream-cdr(stream1), stream-cdr(stream2)))
    )

-- fibs = [0, 1, 1, 2, 3, 5, 8, 13, 21, ...]
-- Each element is the sum of the two previous elements
-- Defined implicitly as a fixed point
```

### Primes via Sieve (Self-Reference)

```
primes = sieve(integers-starting-from(2))

sieve(stream)
  return cons-stream(
    stream-car(stream),
    delay(sieve(
      stream-filter(
        lambda x: x mod stream-car(stream) != 0,
        stream-cdr(stream)
      )
    ))
  )

-- primes references itself indirectly through sieve
-- Each application of sieve removes multiples of the current prime
-- Infinite stream of all primes, computed on demand
```

### Integrating Streams (Calculus Example)

```
integrate-series(stream_y, stream_dy, initial_value, dt)
  return cons-stream(
    initial_value,
    delay(integrate-series(
      stream-add(stream_y, stream-map(lambda dy: dy * dt, stream_dy)),
      stream_dy,
      initial_value + stream-car(stream_dy) * dt,
      dt
    ))
  )

-- Implicit definition: next state computed from current state and derivative
-- Models continuous systems with discrete approximations
```

---

## Exploiting the Stream Paradigm

### Modeling Discrete Events

```
-- Simulate a bouncing ball with implicit physics
position = cons-stream(
  100,  -- Initial position
  delay(stream-map(
    lambda v: position + v,  -- Self-reference
    velocity
  ))
)

velocity = cons-stream(
  0,    -- Initial velocity
  delay(stream-map(
    lambda _: -9.8,  -- Gravity
    acceleration
  ))
)

-- Separate concerns: position defined in terms of velocity
-- Velocity defined in terms of acceleration
-- Enables modular physics simulation
```

### Signal Processing

```
-- Model electrical signal as stream of samples
signal = sample-audio(audio-device, sample-rate)

-- Process without buffering entire audio
filtered = stream-filter(is-above-threshold, signal)
amplified = stream-map(lambda x: x * gain, filtered)

-- Extract features on demand
peaks = find-peaks(amplified)

-- Stream-based pipeline never loads entire audio into memory
```

### Numerical Integration

```
-- Integrate function f over interval [a, b] with step dx
newton-raphson-stream(x_guess, f, f-prime, tolerance)
  return cons-stream(
    x_guess,
    delay(newton-raphson-stream(
      x_guess - f(x_guess) / f-prime(x_guess),  -- Newton iteration
      f,
      f-prime,
      tolerance
    ))
  )

-- Generates sequence of progressively better root approximations
-- Stop when convergence criterion met
-- Implicit definition: each term feeds into next iteration
```

---

## Pseudocode Examples

### Example 1: Fibonacci Access

```
-- Define infinite Fibonacci stream
fibs = fibonacci-stream()  -- [0, 1, 1, 2, 3, 5, 8, 13, 21, ...]

-- Access 10th Fibonacci number
tenth_fib = stream-ref(fibs, 10)
  → 55

-- Computation flow:
--   1. Request fibs[10]
--   2. Force evaluation of tail thunk
--   3. Recursively force until 10 elements deep
--   4. Compute only necessary values: 0→1→1→2→3→5→8→13→21→34→55
--   5. Return 55

-- Memory: Only ~11 values held at once, not all Fibonacci numbers
```

### Example 2: Prime Filtering

```
-- Get first 10 primes
primes = sieve(integers-starting-from(2))
first_ten_primes = stream-take(primes, 10)
  → [2, 3, 5, 7, 11, 13, 17, 19, 23, 29]

-- Computation flow:
--   1. sieve checks if 2 is prime (yes)
--   2. Filters remaining integers by divisibility by 2
--   3. From [3, 5, 7, 9, 11, 13, ...], checks 3 is prime (yes)
--   4. Filters by divisibility by 3
--   5. Continues until 10 primes found
--   6. Only checks divisibility up to the 10th prime

-- Contrast: list-based sieve would pre-compute ALL primes up to some limit
```

### Example 3: Mapped Stream Operations

```
-- Generate stream of squares of even numbers
even_squares = stream-map(
  lambda x: x * x,
  stream-filter(
    lambda x: x mod 2 = 0,
    integers-starting-from(1)
  )
)

-- Access 5th even square
fifth = stream-ref(even_squares, 5)
  → 100  -- (5th even is 10, 10² = 100)

-- Computation:
--   1. integers-starting-from(1) creates [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, ...]
--   2. Filter for even: [2, 4, 6, 8, 10, ...]
--   3. Map x²: [4, 16, 36, 64, 100, ...]
--   4. Return element at index 5: 100

-- Each operation lazy: only computes [1,2] → filter [2] → square [4] → [1,4]
--                      then [3,4] → filter [4] → square [16] → [2,16]
--                      then [5,6] → filter [6] → square [36] → [3,36]
--                      ... continues until 6 elements of squared stream exist
```

### Example 4: Infinite Pairs

```
-- Generate pairs (i, j) where i,j are positive integers
-- Enumerate in order of increasing sum
all_pairs = pairs(
  integers-starting-from(1),
  integers-starting-from(1)
)

-- Get first 12 pairs
first_pairs = stream-take(all_pairs, 12)
  → [[1,1], [1,2], [2,1], [1,3], [2,2], [3,1], 
     [1,4], [2,3], [3,2], [4,1], [1,5], [2,4]]

-- Key insight: Diagonal enumeration ensures all pairs reachable
-- Can't just iterate (1,1), (1,2), (1,3), ... never reach (2,1)
-- Stream interleaving achieves fairness
```

---

## Streams vs Lists Comparison

| Aspect | Lists (Eager) | Streams (Lazy) |
|--------|---------------|----------------|
| **Evaluation** | Immediate, all elements | Deferred, on-demand |
| **Memory** | Holds all elements | Holds current + pending |
| **Infinity** | Impossible to represent | Naturally supported |
| **First Element** | `car(list)` (immediate) | `stream-car(stream)` (immediate) |
| **Rest Elements** | `cdr(list)` computed | `stream-cdr(stream)` forces thunk |
| **Construction** | `cons(a, list)` | `cons-stream(a, delay(...))` |
| **Filtering** | Creates new list | Returns lazy stream |
| **Mapping** | Creates new list | Returns lazy stream |
| **Access Pattern** | Sequential from start | Any position (but linear) |
| **Unused Elements** | Computed anyway | Never computed |
| **Use Case** | Bounded, small sequences | Unbounded, large sequences |
| **Implicit Definition** | Circular lists (problematic) | Streams (elegant) |
| **Performance** | Fast random access | Slow random access |
| **Space** | O(n) for n elements | O(1) amortized |

### When to Use Each

**Use Lists:**
- Bounded sequences (known size)
- Frequent random access
- All elements needed
- Small data sets

**Use Streams:**
- Unbounded sequences
- Sequential access pattern
- Large or infinite data
- Decoupling generation from consumption
- Composing operations without materialization

---

## Summary Table

| Concept | Definition | Example | Use |
|---------|-----------|---------|-----|
| **cons-stream** | Create stream with head and delayed tail | `cons-stream(1, delay([2,3,...])))` | Build streams |
| **stream-car** | Get first element (no force) | `stream-car(s) → 1` | Head access |
| **stream-cdr** | Get rest (forces tail thunk) | `stream-cdr(s) → [2,3,...]` | Recursion |
| **delay** | Wrap expression in thunk | `delay(3 + 4)` | Deferred computation |
| **force** | Evaluate thunk | `force(delayed) → 7` | Execute delayed code |
| **stream-map** | Apply function to all elements | `stream-map(x², integers)` | Transform streams |
| **stream-filter** | Keep elements matching predicate | `stream-filter(even?, integers)` | Select elements |
| **stream-ref** | Access nth element | `stream-ref(s, 10)` | Random position |
| **stream-take** | Get first n elements | `stream-take(s, 5)` | Prefix extraction |
| **Implicit Definition** | Stream defined in terms of itself | `fibs = cons-stream(0, ...)` | Self-referential sequences |
| **Infinite Stream** | Unbounded lazy sequence | `integers-starting-from(1)` | Model infinity |
| **Memoization** | Cache forced thunk value | Thunk stores result | Avoid recomputation |

---

## Key Insights

1. **Lazy Evaluation Decouples Generation and Consumption**: Producing and consuming sequences can be independently designed and composed.

2. **Implicit Streams Enable Elegant Solutions**: Self-referential stream definitions elegantly solve problems that would require complex recursion with lists.

3. **Memory Efficiency via Postponement**: By deferring computation, only accessed elements consume memory, enabling infinite sequences.

4. **Composability Without Materialization**: Stream operations compose naturally without creating intermediate full-sequence copies.

5. **Thunks Enable Control Flow**: Delay/force gives programmers fine-grained control over evaluation order and timing.

---

*Based on concepts from "Structure and Interpretation of Computer Programs" by Abelson and Sussman.*
