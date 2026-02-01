# Recursion Patterns

Recursion is a fundamental programming technique where a function solves a problem by calling itself with simpler inputs. Understanding different recursion patterns is essential for writing correct, efficient, and maintainable recursive solutions.

## Linear Recursion vs Iteration

### Linear Recursion (Single Recursive Call)

A function makes a **single recursive call** to itself, processing one element per level.

#### Factorial (Recursive)

```pseudocode
function factorial_recursive(n):
    if n == 0:
        return 1
    else:
        return n * factorial_recursive(n - 1)
```

**Call Chain:**
```
factorial_recursive(5)
→ 5 * factorial_recursive(4)
  → 4 * factorial_recursive(3)
    → 3 * factorial_recursive(2)
      → 2 * factorial_recursive(1)
        → 1 * factorial_recursive(0)
          → 1
        ← 1 * 1 = 1
      ← 2 * 1 = 2
    ← 3 * 2 = 6
  ← 4 * 6 = 24
← 5 * 24 = 120
```

**Characteristics:**
- Single recursive call per invocation
- Builds up a chain of deferred operations
- Linear growth in call stack depth
- Requires O(n) space for call stack (call frames)

#### Factorial (Iterative)

```pseudocode
function factorial_iterative(n):
    accumulator = 1
    counter = 1
    while counter <= n:
        accumulator = accumulator * counter
        counter = counter + 1
    return accumulator
```

**Characteristics:**
- No recursion; uses loop
- Constant space usage (only local variables)
- O(1) space complexity
- Easy to optimize at the hardware level

### Key Difference: Space Complexity

| Aspect | Linear Recursion | Iteration |
|--------|------------------|-----------|
| **Call Stack Growth** | O(n) | O(1) |
| **Each Iteration** | Creates new stack frame | Reuses same variables |
| **Maximum Depth** | n (limited by stack size) | No depth limit |
| **Memory Usage** | Proportional to input size | Constant |

---

## Tree Recursion

### Definition

**Tree recursion** occurs when a function makes **multiple recursive calls** to itself. Each call branches into more calls, creating a tree-shaped execution pattern.

#### Fibonacci (Tree Recursion)

Naive implementation that makes multiple recursive calls:

```pseudocode
function fib_naive(n):
    if n == 0:
        return 0
    else if n == 1:
        return 1
    else:
        return fib_naive(n - 1) + fib_naive(n - 2)
```

**Call Tree for fib_naive(5):**
```
                    fib(5)
                   /      \
              fib(4)        fib(3)
             /      \       /      \
         fib(3)    fib(2)  fib(2)  fib(1)
        /    \     /    \  /    \
      fib(2) fib(1) fib(1) fib(0) fib(1) fib(0)
     /    \
   fib(1) fib(0)
```

**Problem:** Massive redundant computation
- fib(3) computed 2 times
- fib(2) computed 3 times
- fib(1) computed 5 times
- fib(0) computed 3 times

**Time Complexity:** O(φ^n) where φ ≈ 1.618 (exponential)

#### Counting Change

Count the number of ways to make change for a given amount using a given set of coin denominations:

```pseudocode
function count_change(amount, coins):
    if amount == 0:
        return 1  // one way: use no coins
    else if amount < 0 or coins is empty:
        return 0  // no way to make negative amount
    else:
        first_coin = coins[0]
        rest_coins = coins[1:]
        
        return count_change(amount - first_coin, coins) +  // use first coin
               count_change(amount, rest_coins)             // don't use first coin
```

**Tree Structure:**
```
count_change(10, [penny, nickel, dime])
├── count_change(9, [penny, nickel, dime])   // use penny
│   ├── count_change(8, [penny, nickel, dime])
│   │   └── ...
│   └── count_change(9, [nickel, dime])
│       ├── count_change(4, [nickel, dime])
│       │   └── ...
│       └── count_change(9, [dime])
│           └── ...
└── count_change(10, [nickel, dime])         // don't use penny
    ├── count_change(5, [nickel, dime])
    │   └── ...
    └── count_change(10, [dime])
        └── ...
```

**Characteristics:**
- Two or more recursive calls
- Exponential branching
- Heavy redundant computation without memoization
- Natural for divide-and-conquer problems

### Optimization: Memoization

Cache results to avoid recomputation:

```pseudocode
function count_change_memo(amount, coins):
    memo = empty_cache
    
    function helper(amount, coin_index):
        if amount == 0:
            return 1
        else if amount < 0 or coin_index >= coins.length:
            return 0
        
        if (amount, coin_index) in memo:
            return memo[(amount, coin_index)]
        
        first_coin = coins[coin_index]
        
        result = helper(amount - first_coin, coin_index) +  // use coin
                 helper(amount, coin_index + 1)              // skip coin
        
        memo[(amount, coin_index)] = result
        return result
    
    return helper(amount, 0)
```

**With Memoization:**
- Time Complexity: O(amount × number_of_coins)
- Each subproblem solved exactly once
- Space: O(amount × number_of_coins) for cache

---

## Tail Recursion and Tail-Call Optimization

### Definition

A **tail-recursive** function is one where the recursive call is the **last operation** performed before returning. The result of the recursive call is returned directly, with no further computation.

#### Factorial (Tail-Recursive)

```pseudocode
function factorial_tail(n, accumulator = 1):
    if n == 0:
        return accumulator
    else:
        return factorial_tail(n - 1, n * accumulator)
```

**Call Chain:**
```
factorial_tail(5, 1)
→ factorial_tail(4, 5)
→ factorial_tail(3, 20)
→ factorial_tail(2, 60)
→ factorial_tail(1, 120)
→ factorial_tail(0, 120)
→ 120
```

**Key Property:** Each call immediately returns the result of the next call. No pending operations.

### Tail-Call Optimization (TCO)

Many compilers/interpreters recognize tail recursion and optimize it:

**Without TCO (Stack grows):**
```
Stack Frame: factorial_tail(5, 1)
  Stack Frame: factorial_tail(4, 5)
    Stack Frame: factorial_tail(3, 20)
      Stack Frame: factorial_tail(2, 60)
        Stack Frame: factorial_tail(1, 120)
          Stack Frame: factorial_tail(0, 120)
          ← Return 120
        ← Return 120
      ← Return 120
    ← Return 120
  ← Return 120
← Return 120
```

**With TCO (Stack reused):**
```
Stack Frame: factorial_tail(5, 1)
  [Reused] factorial_tail(4, 5)
    [Reused] factorial_tail(3, 20)
      [Reused] factorial_tail(2, 60)
        [Reused] factorial_tail(1, 120)
          [Reused] factorial_tail(0, 120)
          ← Return 120
        ← Return 120 (same frame)
      ← Return 120 (same frame)
    ← Return 120 (same frame)
  ← Return 120 (same frame)
← Return 120 (same frame)
```

The compiler realizes it can **reuse the same stack frame** for each call.

### Converting to Tail Recursion

**Strategy:** Use an **accumulator** parameter to carry intermediate results.

#### Linear Recursion → Tail Recursion

**Before (Linear):**
```pseudocode
function sum(list):
    if list is empty:
        return 0
    else:
        return first(list) + sum(rest(list))
```

**After (Tail):**
```pseudocode
function sum(list, accumulator = 0):
    if list is empty:
        return accumulator
    else:
        new_acc = accumulator + first(list)
        return sum(rest(list), new_acc)
```

#### Requirements for Tail Recursion

1. Recursive call is the **last statement**
2. Result of recursive call is **returned directly**
3. No operations after the recursive call
4. Accumulator carries all necessary state

### Language Support for TCO

| Language | TCO Support | Notes |
|----------|------------|-------|
| **Scheme** | Yes (guaranteed) | Recursive calls compile to jumps |
| **Functional Languages** (ML, Haskell, Scala) | Yes | Enabled by default |
| **JavaScript** | Partial | Only in strict mode (ES6+); not all engines |
| **Python** | No | Guido van Rossum deliberately disabled |
| **Java** | No | Language design choice |
| **C/C++** | Optional | Some compilers with `-O` flag |

---

## Mutual Recursion

### Definition

Two or more functions call each other recursively. The call graph contains cycles.

#### Even? / Odd?

Determine if a number is even or odd using mutual recursion:

```pseudocode
function is_even(n):
    if n == 0:
        return true
    else:
        return is_odd(n - 1)

function is_odd(n):
    if n == 0:
        return false
    else:
        return is_even(n - 1)
```

**Execution for is_even(4):**
```
is_even(4)
  → is_odd(3)
    → is_even(2)
      → is_odd(1)
        → is_even(0)
          → return true
        ← true
      ← true
    ← true
  ← true
← true
```

**Execution for is_odd(4):**
```
is_odd(4)
  → is_even(3)
    → is_odd(2)
      → is_even(1)
        → is_odd(0)
          → return false
        ← false
      ← false
    ← false
  ← false
← false
```

### Another Example: Abstract Syntax Tree Evaluation

Process an expression tree where numbers and operators call each other:

```pseudocode
function eval_expression(expr):
    if expr is a number:
        return expr
    else if expr is a unary_operation:
        operand = eval_operand(expr.operand)
        return apply_unary_op(expr.operator, operand)
    else if expr is a binary_operation:
        left = eval_operand(expr.left)
        right = eval_operand(expr.right)
        return apply_binary_op(expr.operator, left, right)

function eval_operand(operand):
    return eval_expression(operand)
```

### Use Cases for Mutual Recursion

- **Grammar parsing:** Different rules call each other
- **State machines:** States transition to each other via function calls
- **Language evaluation:** Different expression types evaluate recursively
- **Game trees:** Alternating player moves as mutually recursive functions

---

## Space Complexity Comparison

Comparing space usage (call stack depth) for different recursion patterns:

| Pattern | Example | Depth | Space | Notes |
|---------|---------|-------|-------|-------|
| **Linear Recursion** | factorial_recursive(n) | O(n) | O(n) | Stack grows linearly |
| **Tail Recursion (TCO enabled)** | factorial_tail(n) | O(n) input | O(1) | Stack frame reused |
| **Tail Recursion (TCO disabled)** | factorial_tail(n) | O(n) | O(n) | Compiler doesn't optimize |
| **Tree Recursion (naive)** | fib_naive(n) | O(n) max depth | O(n) | Single path deepest |
| **Tree Recursion (memoized)** | fib_memo(n) | O(n) | O(n) + cache |Cache adds space |
| **Iterative** | factorial_iterative(n) | 1 (no recursion) | O(1) | Only loop variables |
| **Mutual Recursion** | is_even(n) | O(n) | O(n) | Chains between functions |

---

## When to Use Each Pattern

### Use Linear Recursion When:

✓ Natural problem decomposition into single subproblems  
✓ Clarity and elegance matter more than performance  
✓ Input size is small (n < 1000)  
✓ Examples: tree traversal, file system exploration  

**Avoid if:**
- Input size is large
- Deep recursion risks stack overflow
- Performance is critical

### Use Tail Recursion When:

✓ Problem naturally decomposes into subproblems  
✓ Language supports TCO (Scheme, functional languages)  
✓ Need to avoid stack buildup  
✓ Can refactor using accumulators  

**Example languages:** Scheme, Scala, functional languages

**Avoid if:**
- Language doesn't support TCO
- Accumulator pattern becomes convoluted

### Use Tree Recursion When:

✓ Problem has multiple independent subproblems  
✓ Without memoization, is acceptable for small inputs  
✓ Natural divide-and-conquer structure  
✓ Examples: merge sort, quick sort, counting change  

**With Memoization:**
- Dynamic programming opportunities
- Overlapping subproblems
- Transform exponential to polynomial time

### Use Iteration When:

✓ Performance is critical  
✓ Cannot use tail recursion optimization  
✓ Need guaranteed O(1) stack space  
✓ Problem easily expressed as a loop  
✓ Language doesn't support TCO well  

**Examples:** Processing arrays, accumulating results, scanning sequences

### Use Mutual Recursion When:

✓ Problem naturally has circular dependencies  
✓ Multiple types of entities call each other  
✓ Grammar/language parsing  
✓ State machines with alternating transitions  

**Be careful with:** Stack overflow if cycles not bounded; test base cases thoroughly

---

## Converting Recursion to Iteration

### Pattern 1: Linear Recursion → Iteration with Loop

**Recursive:**
```pseudocode
function process_list(items):
    if items is empty:
        return base_result
    else:
        current = first(items)
        result = process_item(current)
        return combine(result, process_list(rest(items)))
```

**Iterative:**
```pseudocode
function process_list(items):
    result = base_result
    for item in items:
        result = combine(result, process_item(item))
    return result
```

### Pattern 2: Tail Recursion → Iteration with Accumulator

**Tail Recursive:**
```pseudocode
function sum_tail(list, acc = 0):
    if list is empty:
        return acc
    else:
        return sum_tail(rest(list), acc + first(list))
```

**Iterative:**
```pseudocode
function sum_iterative(list):
    acc = 0
    for item in list:
        acc = acc + item
    return acc
```

### Pattern 3: Tree Recursion → Iteration with Stack

**Tree Recursive (depth-first):**
```pseudocode
function traverse_tree(node):
    if node is null:
        return
    process(node)
    traverse_tree(node.left)
    traverse_tree(node.right)
```

**Iterative (explicit stack):**
```pseudocode
function traverse_tree_iterative(root):
    if root is null:
        return
    
    stack = new Stack()
    stack.push(root)
    
    while stack is not empty:
        node = stack.pop()
        process(node)
        
        if node.right is not null:
            stack.push(node.right)
        if node.left is not null:
            stack.push(node.left)
```

### Pattern 4: Mutual Recursion → State Machine Loop

**Mutual Recursive:**
```pseudocode
function even(n):
    if n == 0: return true
    else: return odd(n - 1)

function odd(n):
    if n == 0: return false
    else: return even(n - 1)
```

**Iterative (state machine):**
```pseudocode
function is_even_iterative(n):
    state = "even"
    counter = n
    
    while counter > 0:
        state = "odd" if state == "even" else "even"
        counter = counter - 1
    
    return state == "even"
```

---

## Pseudocode Examples Summary

### 1. Linear Recursion: Factorial

```pseudocode
function factorial(n):
    if n == 0 or n == 1:
        return 1
    else:
        return n * factorial(n - 1)
```

**Time:** O(n)  
**Space:** O(n) call stack

---

### 2. Tail Recursion: Factorial with Accumulator

```pseudocode
function factorial_tail(n, acc = 1):
    if n == 0 or n == 1:
        return acc
    else:
        return factorial_tail(n - 1, n * acc)
```

**Time:** O(n)  
**Space:** O(1) with TCO, O(n) without

---

### 3. Tree Recursion: Fibonacci

```pseudocode
function fib(n):
    if n == 0:
        return 0
    else if n == 1:
        return 1
    else:
        return fib(n - 1) + fib(n - 2)
```

**Time:** O(2^n) exponential  
**Space:** O(n) max call depth

---

### 4. Tree Recursion with Memoization: Fibonacci

```pseudocode
function fib_memo(n, cache = {}):
    if n in cache:
        return cache[n]
    
    if n == 0:
        return 0
    else if n == 1:
        return 1
    else:
        result = fib_memo(n - 1, cache) + fib_memo(n - 2, cache)
        cache[n] = result
        return result
```

**Time:** O(n)  
**Space:** O(n) cache + O(n) call stack

---

### 5. Mutual Recursion: Even/Odd

```pseudocode
function is_even(n):
    if n == 0:
        return true
    else:
        return is_odd(n - 1)

function is_odd(n):
    if n == 0:
        return false
    else:
        return is_even(n - 1)
```

**Time:** O(n)  
**Space:** O(n) call stack

---

### 6. Iteration: Factorial

```pseudocode
function factorial_iter(n):
    product = 1
    for counter from 1 to n:
        product = product * counter
    return product
```

**Time:** O(n)  
**Space:** O(1) constant

---

### 7. Counting Change (Tree Recursion)

```pseudocode
function count_change(amount, coins):
    if amount == 0:
        return 1
    else if amount < 0 or coins is empty:
        return 0
    else:
        first = coins[0]
        rest = coins[1:]
        return count_change(amount - first, coins) + 
               count_change(amount, rest)
```

**Time:** O(2^n) exponential without memoization  
**Space:** O(amount × coins) with memoization

---

## Quick Reference Table

| Pattern | Recursion Type | Use Case | Time | Space | Notes |
|---------|---|---|---|---|---|
| **Linear** | Single call | Sequential processing | O(n) | O(n) | Clear but stack-heavy |
| **Tail** | Single call (optimized) | Accumulators, folds | O(n) | O(1)* | *with TCO |
| **Tree** | Multiple calls | Divide-and-conquer | O(2^n) | O(depth) | Use memoization |
| **Tree (memoized)** | Multiple calls (cached) | DP problems | O(n·m) | O(n·m) | Best for overlaps |
| **Mutual** | Circular calls | Parsers, state machines | Varies | Varies | Test base cases |
| **Iterative** | No recursion | When recursion doesn't fit | O(n) | O(1) | Always safe option |

---

## Summary

**Recursion Patterns Table:**

| Pattern | Best For | Pros | Cons |
|---------|----------|------|------|
| **Linear Recursion** | Tree/graph traversal, sequential decomposition | Elegant, natural | O(n) space, stack overflow risk |
| **Tail Recursion** | Loop-like operations, accumulators | Can be optimized to O(1) space | Requires TCO support, less intuitive |
| **Tree Recursion** | Divide-and-conquer, optimization problems | Natural problem decomposition | Exponential time without memoization |
| **Mutual Recursion** | Parsers, alternating processes | Natural for state-dependent logic | Complex to debug, circular dependencies |
| **Iteration** | Performance-critical code, guaranteed stack safety | O(1) space, predictable | Less elegant for recursive problems |

**Key Takeaways:**

1. **Linear recursion** is natural but uses O(n) stack space
2. **Tail recursion** can be optimized to O(1) space if your language supports TCO
3. **Tree recursion** needs memoization to avoid exponential blowup
4. **Iteration** is always a safe, space-efficient fallback
5. **Mutual recursion** is powerful for circular dependencies but requires careful base cases
6. Always consider **space complexity**, not just time complexity
7. Profile your language's **TCO support** before relying on it

---

*Based on concepts from "Structure and Interpretation of Computer Programs" by Abelson and Sussman.*
