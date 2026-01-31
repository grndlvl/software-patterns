# Fenwick Tree (Binary Indexed Tree)

## Overview

A Fenwick tree (also called Binary Indexed Tree or BIT) is a data structure that efficiently supports prefix sum queries and point updates. It achieves O(log n) for both operations using a clever representation based on binary numbers. Fenwick trees are simpler to implement than segment trees while being equally efficient for prefix-based operations.

## Properties

- **Array-based**: Stored in a simple 1-indexed array
- **Binary representation**: Uses least significant bit for indexing
- **Prefix sums**: Naturally computes cumulative sums
- **Space efficient**: Uses exactly n+1 elements
- **1-indexed**: Typically uses indices 1 to n

### Key Insight

The tree structure is implicit in the binary representation of indices:
- Each index i is responsible for elements from `i - LSB(i) + 1` to `i`
- LSB(i) = `i & (-i)` gives the least significant bit

## Time Complexity

| Operation      | Complexity |
|----------------|------------|
| Build          | O(n)       |
| Point Update   | O(log n)   |
| Prefix Query   | O(log n)   |
| Range Query    | O(log n)   |
| Point Query    | O(log n)   |

## Space Complexity

O(n) - exactly n+1 elements (index 0 unused in standard implementation).

## Operations

### Least Significant Bit

```pseudocode
function lowbit(x) {
    return x & (-x)
}

// Examples:
// lowbit(6)  = lowbit(110)  = 2  (010)
// lowbit(12) = lowbit(1100) = 4  (100)
// lowbit(8)  = lowbit(1000) = 8  (1000)
```

### Point Update

Add a value to an element:

```pseudocode
function update(index, delta) {
    while index <= n {
        tree[index] = tree[index] + delta
        index = index + lowbit(index)  // Move to parent
    }
}
```

### Prefix Sum Query

Sum of elements from 1 to index:

```pseudocode
function prefixSum(index) {
    sum = 0
    while index > 0 {
        sum = sum + tree[index]
        index = index - lowbit(index)  // Move to previous range
    }
    return sum
}
```

### Range Sum Query

Sum of elements from left to right:

```pseudocode
function rangeSum(left, right) {
    return prefixSum(right) - prefixSum(left - 1)
}
```

### Build

```pseudocode
// Method 1: O(n log n) - simple
function build(arr) {
    tree = new Array(n + 1).fill(0)
    for i from 1 to n {
        update(i, arr[i])
    }
}

// Method 2: O(n) - optimized
function buildOptimized(arr) {
    tree = new Array(n + 1)
    tree[0] = 0

    for i from 1 to n {
        tree[i] = arr[i]
    }

    for i from 1 to n {
        parent = i + lowbit(i)
        if parent <= n {
            tree[parent] = tree[parent] + tree[i]
        }
    }
}
```

## Implementation

### Basic Fenwick Tree (Sum)

```pseudocode
class FenwickTree {
    tree[]
    n

    function constructor(size) {
        this.n = size
        this.tree = new Array(this.n + 1).fill(0)
    }

    function constructorFromArray(arr) {
        this.n = arr.length
        this.tree = new Array(this.n + 1).fill(0)

        // Copy array (1-indexed)
        for i from 0 to arr.length - 1 {
            this.tree[i + 1] = arr[i]
        }

        // Build tree in O(n)
        for i from 1 to this.n {
            parent = i + (i & (-i))
            if parent <= this.n {
                this.tree[parent] = this.tree[parent] + this.tree[i]
            }
        }
    }

    function update(index, delta) {
        while index <= this.n {
            this.tree[index] = this.tree[index] + delta
            index = index + (index & (-index))
        }
    }

    function prefixSum(index) {
        sum = 0
        while index > 0 {
            sum = sum + this.tree[index]
            index = index - (index & (-index))
        }
        return sum
    }

    function rangeSum(left, right) {
        return prefixSum(right) - prefixSum(left - 1)
    }

    // Get single element value
    function get(index) {
        return rangeSum(index, index)
    }

    // Set element to specific value
    function set(index, value) {
        current = get(index)
        update(index, value - current)
    }
}
```

### Range Update, Point Query

For scenarios where you update ranges but query single points:

```pseudocode
class RangeUpdateFenwick {
    tree[]
    n

    function constructor(size) {
        this.n = size
        this.tree = new Array(this.n + 1).fill(0)
    }

    // Add delta to all elements in [left, right]
    function rangeUpdate(left, right, delta) {
        update(left, delta)
        update(right + 1, -delta)
    }

    function update(index, delta) {
        while index <= this.n {
            this.tree[index] = this.tree[index] + delta
            index = index + (index & (-index))
        }
    }

    // Get value at index
    function pointQuery(index) {
        sum = 0
        while index > 0 {
            sum = sum + this.tree[index]
            index = index - (index & (-index))
        }
        return sum
    }
}
```

### Range Update, Range Query

Uses two Fenwick trees:

```pseudocode
class RangeRangeFenwick {
    tree1[]  // Stores differences
    tree2[]  // Stores i * diff for proper range sums
    n

    function constructor(size) {
        this.n = size
        this.tree1 = new Array(this.n + 1).fill(0)
        this.tree2 = new Array(this.n + 1).fill(0)
    }

    function update(tree, index, delta) {
        while index <= this.n {
            tree[index] = tree[index] + delta
            index = index + (index & (-index))
        }
    }

    function query(tree, index) {
        sum = 0
        while index > 0 {
            sum = sum + tree[index]
            index = index - (index & (-index))
        }
        return sum
    }

    function rangeUpdate(left, right, delta) {
        update(this.tree1, left, delta)
        update(this.tree1, right + 1, -delta)
        update(this.tree2, left, delta * (left - 1))
        update(this.tree2, right + 1, -delta * right)
    }

    function prefixSum(index) {
        return query(this.tree1, index) * index - query(this.tree2, index)
    }

    function rangeSum(left, right) {
        return prefixSum(right) - prefixSum(left - 1)
    }
}
```

### 2D Fenwick Tree

```pseudocode
class FenwickTree2D {
    tree[][]
    rows
    cols

    function constructor(rows, cols) {
        this.rows = rows
        this.cols = cols
        this.tree = new Array(rows + 1)
        for i from 0 to rows {
            this.tree[i] = new Array(cols + 1).fill(0)
        }
    }

    function update(row, col, delta) {
        i = row
        while i <= this.rows {
            j = col
            while j <= this.cols {
                this.tree[i][j] = this.tree[i][j] + delta
                j = j + (j & (-j))
            }
            i = i + (i & (-i))
        }
    }

    function prefixSum(row, col) {
        sum = 0
        i = row
        while i > 0 {
            j = col
            while j > 0 {
                sum = sum + this.tree[i][j]
                j = j - (j & (-j))
            }
            i = i - (i & (-i))
        }
        return sum
    }

    // Sum of rectangle from (r1,c1) to (r2,c2)
    function rangeSum(r1, c1, r2, c2) {
        return prefixSum(r2, c2)
             - prefixSum(r1 - 1, c2)
             - prefixSum(r2, c1 - 1)
             + prefixSum(r1 - 1, c1 - 1)
    }
}
```

## Classic Applications

### Count Inversions

```pseudocode
function countInversions(arr) {
    // Coordinate compression
    sorted = sort(copy(arr))
    rank = new Map()
    for i from 0 to sorted.length - 1 {
        rank.set(sorted[i], i + 1)
    }

    bit = new FenwickTree(arr.length)
    inversions = 0

    // Process from right to left
    for i from arr.length - 1 down to 0 {
        r = rank.get(arr[i])
        inversions = inversions + bit.prefixSum(r - 1)  // Count smaller elements to the right
        bit.update(r, 1)
    }

    return inversions
}
```

### Range Frequency Query

```pseudocode
function countInRange(arr, queries) {
    // For each unique value, store positions in BIT
    // Can answer: how many times does value v appear in range [l, r]?
}
```

### Find k-th Smallest

```pseudocode
function findKth(bit, k) {
    pos = 0
    sum = 0
    logN = floor(log2(bit.n))

    for i from logN down to 0 {
        nextPos = pos + (1 << i)
        if nextPos <= bit.n and sum + bit.tree[nextPos] < k {
            sum = sum + bit.tree[nextPos]
            pos = nextPos
        }
    }

    return pos + 1
}
```

## Use Cases

- **Prefix sums with updates**: Running totals
- **Inversion counting**: Merge sort alternative
- **Range frequency queries**: Count occurrences
- **Order statistics**: k-th element queries
- **Coordinate compression**: Handling large value ranges
- **2D queries**: Matrix region sums

## Advantages

- **Simple implementation**: ~10 lines of code
- **Space efficient**: Exactly n+1 elements
- **Cache friendly**: Sequential memory access patterns
- **Easy to extend**: 2D, range update variants
- **Low constant factor**: Very fast in practice

## Disadvantages

- **Limited operations**: Best for prefix sums
- **1-indexed**: Can be confusing
- **Not intuitive**: Binary representation not obvious
- **Limited flexibility**: Segment tree handles more cases
- **No lazy propagation**: Range-range requires two trees

## Comparison with Alternatives

| Aspect          | Fenwick Tree | Segment Tree | Prefix Array | Sqrt Decomp |
|-----------------|--------------|--------------|--------------|-------------|
| Build           | O(n)         | O(n)         | O(n)         | O(n)        |
| Point update    | O(log n)     | O(log n)     | O(n)         | O(1)        |
| Prefix query    | O(log n)     | O(log n)     | O(1)         | O(√n)       |
| Range query     | O(log n)     | O(log n)     | O(1)         | O(√n)       |
| Range update    | O(log n)*    | O(log n)     | O(n)         | O(√n)       |
| Space           | O(n)         | O(n)         | O(n)         | O(n)        |
| Implementation  | Simple       | Medium       | Trivial      | Simple      |
| Flexibility     | Limited      | High         | None         | Medium      |

*With special range update variant

## Common Pitfalls

- **0-indexed confusion**: Fenwick trees are 1-indexed
- **Update vs set**: update() adds delta, not sets value
- **Off-by-one**: rangeSum(l, r) needs prefixSum(l-1)
- **Overflow**: Sum can overflow for large values
- **Negative indices**: lowbit doesn't work for index 0
- **Build complexity**: Naive build is O(n log n), not O(n)

## Related Structures

- **Segment Tree**: More flexible but more complex
- **Sparse Table**: O(1) query but no updates
- **Prefix Sum Array**: O(1) query but O(n) update
- **2D Fenwick Tree**: For matrix operations
- **Persistent Fenwick**: Version history support

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
