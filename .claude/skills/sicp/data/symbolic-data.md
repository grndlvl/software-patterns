# Symbolic Data: Quotation and Abstraction

Symbolic data represents non-numeric information using data structures. This module explores manipulating symbols, expressions, and abstract data as computational objects.

## Core Concept: Quotation

Quotation distinguishes between:
- **Symbols as values**: `'a` represents the symbol "a" itself
- **Symbols as evaluation**: `a` evaluates to the value bound to variable a

### Quotation Examples

```
Expression          | Type              | Meaning
--------------------|-------------------|------------------------------------------
'a                  | Quoted symbol     | The symbol "a" as data
(quote a)           | Quote form        | Same as 'a
a                   | Variable ref      | Evaluates variable a
'(a b c)            | Quoted list       | List of symbols: [a, b, c]
(list 'a 'b 'c)     | Constructed list  | List of 3 symbol values
```

**Why quotation matters**: Without it, `(a b c)` would attempt to call `a` as a function.

---

## Symbolic Differentiation

A classic example: automatically compute derivatives using symbolic manipulation.

### Mathematical Rules

```
Rule 1: d/dx(c) = 0           [constant]
Rule 2: d/dx(x) = 1           [variable]
Rule 3: d/dx(u + v) = du/dx + dv/dx
Rule 4: d/dx(u * v) = u*(dv/dx) + v*(du/dx)
```

### Pseudocode Implementation

```pseudocode
function differentiate(expr, var) {
    if (isNumber(expr)) {
        return 0
    }
    
    if (isVariable(expr)) {
        return (expr == var) ? 1 : 0
    }
    
    if (isSum(expr)) {
        let [u, v] = operands(expr)
        return makeSum(
            differentiate(u, var),
            differentiate(v, var)
        )
    }
    
    if (isProduct(expr)) {
        let [u, v] = operands(expr)
        return makeSum(
            makeProduct(u, differentiate(v, var)),
            makeProduct(v, differentiate(u, var))
        )
    }
}

// Constructors
function makeSum(a, b) {
    if (isNumber(a) && a == 0) return b
    if (isNumber(b) && b == 0) return a
    if (isNumber(a) && isNumber(b)) return a + b
    return list('+', a, b)
}

function makeProduct(a, b) {
    if (isNumber(a) && a == 0) return 0
    if (isNumber(a) && a == 1) return b
    if (isNumber(b) && b == 0) return 0
    if (isNumber(b) && b == 1) return a
    if (isNumber(a) && isNumber(b)) return a * b
    return list('*', a, b)
}

// Selectors
function isSum(expr) {
    return isTagged(expr, '+')
}

function isProduct(expr) {
    return isTagged(expr, '*')
}

function operands(expr) {
    return tail(expr)  // All elements after operator
}

function isTagged(expr, tag) {
    return isSequence(expr) && 
           head(expr) == tag
}
```

### Example Trace

```
Differentiate: (x + 3) with respect to x

Step 1: Recognize as sum
Step 2: Differentiate x → 1
Step 3: Differentiate 3 → 0
Step 4: makeSum(1, 0) → 1

Result: 1 ✓
```

---

## Representing Sets

Different data structures for sets with trade-offs:

### 1. Unordered Lists

```pseudocode
// Simple list representation
SET: [2, 3, 5, 7]  // Order irrelevant

function elementOfSet(elem, set) {
    for item in set {
        if item == elem return true
    }
    return false
}

function adjoin(elem, set) {
    if elementOfSet(elem, set) {
        return set
    }
    return append([elem], set)
}

function union(set1, set2) {
    result = set1
    for elem in set2 {
        if not elementOfSet(elem, result) {
            result = adjoin(elem, result)
        }
    }
    return result
}
```

**Complexity:**
| Operation | Time |
|-----------|------|
| Element lookup | O(n) |
| Insert | O(n) |
| Union | O(n²) |

---

### 2. Ordered Lists

```pseudocode
// Sorted list for faster search
SET: [2, 3, 5, 7, 11]  // Elements sorted

function elementOfSet(elem, set) {
    if empty(set) return false
    
    let first = head(set)
    if elem == first return true
    if elem < first return false  // Not found, would be here
    
    return elementOfSet(elem, tail(set))
}

function adjoin(elem, set) {
    if empty(set) return [elem]
    
    let first = head(set)
    if elem == first return set
    if elem < first return cons(elem, set)
    
    return cons(first, adjoin(elem, tail(set)))
}

function intersection(set1, set2) {
    result = []
    for elem in set1 {
        if elementOfSet(elem, set2) {
            result = adjoin(elem, result)
        }
    }
    return result
}
```

**Complexity:**
| Operation | Time |
|-----------|------|
| Element lookup | O(n) |
| Insert | O(n) |
| Intersection | O(n²) |

---

### 3. Binary Search Trees

```pseudocode
// Tree structure for fast lookup
NODE: {
    value: data,
    left: subtree,
    right: subtree
}

function elementOfSet(elem, tree) {
    if tree == null return false
    
    let nodeValue = value(tree)
    if elem == nodeValue return true
    if elem < nodeValue {
        return elementOfSet(elem, leftBranch(tree))
    } else {
        return elementOfSet(elem, rightBranch(tree))
    }
}

function adjoin(elem, tree) {
    if tree == null return makeTree(elem, null, null)
    
    let nodeValue = value(tree)
    if elem == nodeValue return tree
    if elem < nodeValue {
        return makeTree(
            nodeValue,
            adjoin(elem, leftBranch(tree)),
            rightBranch(tree)
        )
    } else {
        return makeTree(
            nodeValue,
            leftBranch(tree),
            adjoin(elem, rightBranch(tree))
        )
    }
}

function makeTree(value, left, right) {
    return {
        value: value,
        left: left,
        right: right
    }
}
```

**Complexity (balanced tree):**
| Operation | Time |
|-----------|------|
| Element lookup | O(log n) |
| Insert | O(log n) |
| Unbalanced worst case | O(n) |

---

## Set Operations Comparison

### Union

**Unordered List:**
```pseudocode
function union(s1, s2) {
    result = s1
    for elem in s2 {
        if not elementOfSet(elem, result) {
            result = adjoin(elem, result)
        }
    }
    return result
}
// Time: O(n²) - checks each element of s2 against s1
```

**Ordered List:**
```pseudocode
function union(s1, s2) {
    if empty(s1) return s2
    if empty(s2) return s1
    
    let first1 = head(s1)
    let first2 = head(s2)
    
    if first1 == first2 {
        return cons(first1, union(tail(s1), tail(s2)))
    } else if first1 < first2 {
        return cons(first1, union(tail(s1), s2))
    } else {
        return cons(first2, union(s1, tail(s2)))
    }
}
// Time: O(n) - linear merge
```

**Binary Tree:**
```pseudocode
function union(t1, t2) {
    if t1 == null return t2
    return union(
        leftBranch(t1),
        union(rightBranch(t1), adjoin(value(t1), t2))
    )
}
// Time: O(n log n) average, O(n²) worst case
```

---

## Huffman Encoding Trees

Tree representation for variable-length binary codes.

### Tree Structure

```
A Huffman tree for symbols with frequencies:
A:5, B:9, C:12, D:16, E:45

                    100
                   /    \
                  45     55
                 /      /  \
                E      25   30
                      / \  / \
                     B  12 C  D
                         
Codes (path from root):
E: 0         (1 bit)
D: 101       (3 bits)
C: 100       (3 bits)
A: 1101      (4 bits)
B: 1100      (4 bits)
```

### Tree Construction

```pseudocode
// Priority queue of (frequency, tree) pairs

function buildHuffmanTree(frequencies) {
    // frequencies: list of [symbol, freq] pairs
    
    pq = PriorityQueue()
    
    // Insert leaf nodes
    for [symbol, freq] in frequencies {
        leaf = makeLeaf(symbol, freq)
        pq.insert(freq, leaf)
    }
    
    // Repeatedly merge two smallest trees
    while pq.size() > 1 {
        let [freq1, tree1] = pq.extractMin()
        let [freq2, tree2] = pq.extractMin()
        
        mergedFreq = freq1 + freq2
        mergedTree = makeInterior(tree1, tree2)
        
        pq.insert(mergedFreq, mergedTree)
    }
    
    return pq.extractMin()[1]  // Return final tree
}

// Tree node selectors
function isLeaf(node) {
    return length(node) == 2  // [leaf-symbol, frequency]
}

function makeLeaf(symbol, freq) {
    return [symbol, freq]
}

function makeInterior(left, right) {
    return [left, right, freq(left) + freq(right)]
}

function leftBranch(tree) {
    return tree[0]
}

function rightBranch(tree) {
    return tree[1]
}

function symbol(leaf) {
    return leaf[0]
}

function freq(node) {
    return last(node)
}
```

### Encoding and Decoding

```pseudocode
function encode(message, tree) {
    bits = []
    for symbol in message {
        bits.concat(encodeSymbol(symbol, tree))
    }
    return bits
}

function encodeSymbol(symbol, tree) {
    if isLeaf(tree) {
        return []  // Shouldn't search past a leaf
    }
    
    if symbolInTree(symbol, leftBranch(tree)) {
        return [0].concat(encodeSymbol(symbol, leftBranch(tree)))
    } else {
        return [1].concat(encodeSymbol(symbol, rightBranch(tree)))
    }
}

function decode(bits, tree) {
    result = []
    current = tree
    
    for bit in bits {
        if bit == 0 {
            current = leftBranch(current)
        } else {
            current = rightBranch(current)
        }
        
        if isLeaf(current) {
            result.append(symbol(current))
            current = tree
        }
    }
    
    return result
}
```

### Example

```
Message: [A, B, C, A, D]

Encoding:
A → 1101
B → 1100
C → 100
A → 1101
D → 101

Bit sequence: 1101 1100 100 1101 101
Compression: 5 symbols × avg 3.2 bits = 16 bits
vs fixed: 5 symbols × 3 bits = 15 bits (marginal here)
Better with more skewed frequencies
```

---

## Pattern Matching Basics

Matching symbolic expressions against patterns.

### Pattern Language

```pseudocode
// Patterns can contain:
// - Literal symbols: a, b, +, *
// - Wildcards: ?var (matches any single value)
// - Sequence wildcards: ?seq (matches 0+ values)

Pattern           | Expression      | Match | Bindings
------------------|-----------------|-------|------------------
(f ?x ?y)         | (f 3 5)         | Yes   | ?x=3, ?y=5
(f ?x ?y)         | (f 3)           | No    | -
(?op ?x ?y)       | (+ 2 3)         | Yes   | ?op=+, ?x=2, ?y=3
(+ ?x 1)          | (+ 5 1)         | Yes   | ?x=5
(+ ?x 1)          | (+ 5 2)         | No    | -
(* ?x ?x)         | (* 5 5)         | Yes   | ?x=5
(* ?x ?x)         | (* 3 5)         | No    | -
```

### Simple Matcher

```pseudocode
function match(pattern, expr) {
    // Returns bindings dict on success, null on failure
    
    if isVariable(pattern) {
        return {pattern: expr}  // Single binding
    }
    
    if isLiteral(pattern) {
        return pattern == expr ? {} : null
    }
    
    if isSequence(pattern) && isSequence(expr) {
        return matchSeq(pattern, expr)
    }
    
    return null
}

function matchSeq(patterns, exprs) {
    bindings = {}
    
    // Simple case: equal length, no sequence wildcards
    if length(patterns) != length(exprs) {
        return null
    }
    
    for i = 0 to length(patterns) {
        let result = match(patterns[i], exprs[i])
        
        if result == null {
            return null
        }
        
        // Merge bindings, check for conflicts
        for [var, val] in result {
            if bindings[var] exists && bindings[var] != val {
                return null  // Conflict
            }
            bindings[var] = val
        }
    }
    
    return bindings
}

function isVariable(x) {
    return isSymbol(x) && startsWith(x, "?")
}

function isLiteral(x) {
    return isSymbol(x) || isNumber(x)
}
```

### Pattern Matching in Rules

```pseudocode
// Using patterns to write transformation rules

RULES = [
    {
        pattern: (+ ?x 0),
        result: ?x
    },
    {
        pattern: (* ?x 0),
        result: 0
    },
    {
        pattern: (* ?x 1),
        result: ?x
    },
    {
        pattern: (+ ?x ?x),
        result: (* 2 ?x)
    }
]

function simplify(expr) {
    for rule in RULES {
        let bindings = match(rule.pattern, expr)
        
        if bindings != null {
            return substitute(rule.result, bindings)
        }
    }
    
    return expr  // No rule matched
}

function substitute(template, bindings) {
    if isVariable(template) {
        return bindings[template] || template
    }
    
    if isSequence(template) {
        return [substitute(x, bindings) for x in template]
    }
    
    return template
}
```

---

## Pseudocode Examples

### Example 1: Symbolic Sum Simplification

```pseudocode
expr = parse("(+ (* 2 x) (* 3 x))")  // (+ (* 2 x) (* 3 x))

// Recursively walk tree
function simplifyExpr(e) {
    if not isSequence(e) return e
    
    if isSum(e) {
        let [u, v] = operands(e)
        su = simplifyExpr(u)
        sv = simplifyExpr(v)
        return makeSum(su, sv)
    }
    
    if isProduct(e) {
        let [u, v] = operands(e)
        su = simplifyExpr(u)
        sv = simplifyExpr(v)
        return makeProduct(su, sv)
    }
    
    return e
}

result = simplifyExpr(expr)
// Result: (+ (* 2 x) (* 3 x))
// Could add further rules: (+ (* a x) (* b x)) → (* (+ a b) x)
```

### Example 2: Set Membership Across Representations

```pseudocode
// Use abstract selector to handle all representations

function isMember(elem, set) {
    // Dispatch to appropriate implementation
    if isOrderedList(set) {
        return isInOrderedList(elem, set)
    }
    
    if isTree(set) {
        return isInTree(elem, set)
    }
    
    if isUnorderedList(set) {
        return isInUnorderedList(elem, set)
    }
    
    return false
}

// Code using sets stays the same!
mySet = createOrderedListSet([1, 2, 3, 4, 5])
isMember(3, mySet)  // true
```

### Example 3: Quote as Data Constructor

```pseudocode
// Building quoted lists as data

function buildQuotedList(...elements) {
    return quote([elements])  // quote prevents evaluation
}

// Without quote:
// [sqrt, 2] → error: sqrt is undefined variable
// 
// With quote:
// '(sqrt 2) → the symbol list [sqrt, 2]

// Can now manipulate as data
expr = '(+ (* 2 x) 3)
firstOp = head(expr)  // '+
operand1 = second(expr)  // (* 2 x)
operand2 = third(expr)  // 3
```

---

## Representation Comparison Table

| Aspect | Unordered Lists | Ordered Lists | Binary Trees |
|--------|-----------------|---------------|--------------|
| **Element Test** | O(n) | O(n) | O(log n) avg, O(n) worst |
| **Insert** | O(n) | O(n) | O(log n) avg, O(n) worst |
| **Delete** | O(n) | O(n) | O(log n) avg, O(n) worst |
| **Union** | O(n²) | O(n) | O(n log n) avg |
| **Intersection** | O(n²) | O(n) | O(n log n) avg |
| **Space** | O(n) | O(n) | O(n) + balance overhead |
| **Implementation** | Simple | Moderate | Complex |
| **Best Use Case** | Small sets, simple | Moderate size | Large sets, frequent ops |

---

## Summary Table

| Concept | Purpose | Key Operation | Example |
|---------|---------|----------------|---------|
| **Quotation** | Prevent evaluation of symbols | `'x` vs `x` | Treating code as data |
| **Differentiation** | Symbolic algebraic manipulation | `d/dx(expr)` | Computing derivatives |
| **Sets (Unordered)** | Simple symbolic grouping | `elementOfSet(e, s)` | Fast construction |
| **Sets (Ordered)** | Fast lookup with structure | `elementOfSet(e, s)` | Binary search leverage |
| **Sets (Trees)** | Optimal for large sets | `elementOfSet(e, tree)` | Balanced search |
| **Huffman Trees** | Variable-length encoding | `encode(msg, tree)` | Data compression |
| **Pattern Matching** | Symbolic transformation | `match(pattern, expr)` | Rule application |

---

## Key Design Principles

1. **Data Abstraction**: Represent sets abstractly; switch implementations without changing client code
2. **Symbolic Processing**: Manipulate expressions as data, not just values
3. **Quotation**: Use quoting to distinguish code from data
4. **Hierarchical Structure**: Trees represent symbolic dependencies and relationships
5. **Pattern Matching**: Declarative rules replace procedural branching

---

*Based on concepts from "Structure and Interpretation of Computer Programs" by Abelson and Sussman.*
