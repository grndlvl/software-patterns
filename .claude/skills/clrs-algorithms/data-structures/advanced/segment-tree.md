# Segment Tree

## Overview

A segment tree is a binary tree data structure used for storing information about intervals (segments) of an array. It allows efficient querying of aggregate information (sum, minimum, maximum, GCD, etc.) over any range while supporting point or range updates. Segment trees achieve O(log n) for both queries and updates.

## Properties

- **Binary tree structure**: Complete binary tree stored in array
- **Leaf nodes**: Represent individual array elements
- **Internal nodes**: Represent aggregation of child segments
- **Segment coverage**: Each node covers a contiguous range
- **Height**: O(log n) for n elements

### Tree Size

For an array of size n:
- Number of leaves: n
- Total nodes: 2n - 1 (perfect tree) or up to 4n (safe allocation)
- Height: ceil(log2(n))

## Time Complexity

| Operation     | Complexity |
|---------------|------------|
| Build         | O(n)       |
| Point Query   | O(log n)   |
| Range Query   | O(log n)   |
| Point Update  | O(log n)   |
| Range Update  | O(log n)*  |

*With lazy propagation

## Space Complexity

O(n) for the segment tree array (typically allocated as 4n for safety).

## Operations

### Tree Indexing

For 1-indexed array representation:
```
Parent(i) = i / 2
Left Child(i) = 2 * i
Right Child(i) = 2 * i + 1
```

For 0-indexed:
```
Parent(i) = (i - 1) / 2
Left Child(i) = 2 * i + 1
Right Child(i) = 2 * i + 2
```

### Build

```pseudocode
function build(arr) {
    n = arr.length
    tree = new Array(4 * n)  // Safe size allocation

    function buildTree(node, start, end) {
        if start == end {
            // Leaf node
            tree[node] = arr[start]
        } else {
            mid = (start + end) / 2
            leftChild = 2 * node
            rightChild = 2 * node + 1

            buildTree(leftChild, start, mid)
            buildTree(rightChild, mid + 1, end)

            // Internal node = aggregate of children
            tree[node] = combine(tree[leftChild], tree[rightChild])
        }
    }

    buildTree(1, 0, n - 1)
    return tree
}
```

### Point Update

```pseudocode
function pointUpdate(index, value) {
    function update(node, start, end, idx, val) {
        if start == end {
            // Leaf node
            tree[node] = val
        } else {
            mid = (start + end) / 2
            leftChild = 2 * node
            rightChild = 2 * node + 1

            if idx <= mid {
                update(leftChild, start, mid, idx, val)
            } else {
                update(rightChild, mid + 1, end, idx, val)
            }

            // Recalculate internal node
            tree[node] = combine(tree[leftChild], tree[rightChild])
        }
    }

    update(1, 0, n - 1, index, value)
}
```

### Range Query

```pseudocode
function rangeQuery(left, right) {
    function query(node, start, end, l, r) {
        // No overlap
        if r < start or end < l {
            return IDENTITY  // 0 for sum, INF for min, -INF for max
        }

        // Complete overlap
        if l <= start and end <= r {
            return tree[node]
        }

        // Partial overlap
        mid = (start + end) / 2
        leftChild = 2 * node
        rightChild = 2 * node + 1

        leftResult = query(leftChild, start, mid, l, r)
        rightResult = query(rightChild, mid + 1, end, l, r)

        return combine(leftResult, rightResult)
    }

    return query(1, 0, n - 1, left, right)
}
```

## Implementation

### Sum Segment Tree

```pseudocode
class SumSegmentTree {
    tree[]
    n

    function constructor(arr) {
        this.n = arr.length
        this.tree = new Array(4 * this.n).fill(0)
        if this.n > 0 {
            build(arr, 1, 0, this.n - 1)
        }
    }

    function build(arr, node, start, end) {
        if start == end {
            this.tree[node] = arr[start]
        } else {
            mid = (start + end) / 2
            build(arr, 2 * node, start, mid)
            build(arr, 2 * node + 1, mid + 1, end)
            this.tree[node] = this.tree[2 * node] + this.tree[2 * node + 1]
        }
    }

    function update(index, value) {
        updateHelper(1, 0, this.n - 1, index, value)
    }

    function updateHelper(node, start, end, idx, val) {
        if start == end {
            this.tree[node] = val
        } else {
            mid = (start + end) / 2
            if idx <= mid {
                updateHelper(2 * node, start, mid, idx, val)
            } else {
                updateHelper(2 * node + 1, mid + 1, end, idx, val)
            }
            this.tree[node] = this.tree[2 * node] + this.tree[2 * node + 1]
        }
    }

    function query(left, right) {
        return queryHelper(1, 0, this.n - 1, left, right)
    }

    function queryHelper(node, start, end, l, r) {
        if r < start or end < l {
            return 0  // Identity for sum
        }
        if l <= start and end <= r {
            return this.tree[node]
        }

        mid = (start + end) / 2
        leftSum = queryHelper(2 * node, start, mid, l, r)
        rightSum = queryHelper(2 * node + 1, mid + 1, end, l, r)
        return leftSum + rightSum
    }
}
```

### Min/Max Segment Tree

```pseudocode
class MinSegmentTree {
    tree[]
    n
    INF = Infinity

    function constructor(arr) {
        this.n = arr.length
        this.tree = new Array(4 * this.n).fill(this.INF)
        if this.n > 0 {
            build(arr, 1, 0, this.n - 1)
        }
    }

    function build(arr, node, start, end) {
        if start == end {
            this.tree[node] = arr[start]
        } else {
            mid = (start + end) / 2
            build(arr, 2 * node, start, mid)
            build(arr, 2 * node + 1, mid + 1, end)
            this.tree[node] = min(this.tree[2 * node], this.tree[2 * node + 1])
        }
    }

    function update(index, value) {
        updateHelper(1, 0, this.n - 1, index, value)
    }

    function updateHelper(node, start, end, idx, val) {
        if start == end {
            this.tree[node] = val
        } else {
            mid = (start + end) / 2
            if idx <= mid {
                updateHelper(2 * node, start, mid, idx, val)
            } else {
                updateHelper(2 * node + 1, mid + 1, end, idx, val)
            }
            this.tree[node] = min(this.tree[2 * node], this.tree[2 * node + 1])
        }
    }

    function queryMin(left, right) {
        return queryHelper(1, 0, this.n - 1, left, right)
    }

    function queryHelper(node, start, end, l, r) {
        if r < start or end < l {
            return this.INF  // Identity for min
        }
        if l <= start and end <= r {
            return this.tree[node]
        }

        mid = (start + end) / 2
        leftMin = queryHelper(2 * node, start, mid, l, r)
        rightMin = queryHelper(2 * node + 1, mid + 1, end, l, r)
        return min(leftMin, rightMin)
    }
}
```

### Lazy Propagation (Range Updates)

```pseudocode
class LazySegmentTree {
    tree[]
    lazy[]
    n

    function constructor(arr) {
        this.n = arr.length
        this.tree = new Array(4 * this.n).fill(0)
        this.lazy = new Array(4 * this.n).fill(0)
        if this.n > 0 {
            build(arr, 1, 0, this.n - 1)
        }
    }

    function build(arr, node, start, end) {
        if start == end {
            this.tree[node] = arr[start]
        } else {
            mid = (start + end) / 2
            build(arr, 2 * node, start, mid)
            build(arr, 2 * node + 1, mid + 1, end)
            this.tree[node] = this.tree[2 * node] + this.tree[2 * node + 1]
        }
    }

    function pushDown(node, start, end) {
        if this.lazy[node] != 0 {
            mid = (start + end) / 2

            // Apply lazy value to children
            this.tree[2 * node] += this.lazy[node] * (mid - start + 1)
            this.tree[2 * node + 1] += this.lazy[node] * (end - mid)

            // Propagate lazy value
            this.lazy[2 * node] += this.lazy[node]
            this.lazy[2 * node + 1] += this.lazy[node]

            // Clear lazy value
            this.lazy[node] = 0
        }
    }

    function rangeUpdate(left, right, value) {
        updateRange(1, 0, this.n - 1, left, right, value)
    }

    function updateRange(node, start, end, l, r, val) {
        if r < start or end < l {
            return
        }

        if l <= start and end <= r {
            // Complete overlap - apply lazy update
            this.tree[node] += val * (end - start + 1)
            this.lazy[node] += val
            return
        }

        // Partial overlap - push down and recurse
        pushDown(node, start, end)

        mid = (start + end) / 2
        updateRange(2 * node, start, mid, l, r, val)
        updateRange(2 * node + 1, mid + 1, end, l, r, val)

        this.tree[node] = this.tree[2 * node] + this.tree[2 * node + 1]
    }

    function query(left, right) {
        return queryRange(1, 0, this.n - 1, left, right)
    }

    function queryRange(node, start, end, l, r) {
        if r < start or end < l {
            return 0
        }

        if l <= start and end <= r {
            return this.tree[node]
        }

        pushDown(node, start, end)

        mid = (start + end) / 2
        leftSum = queryRange(2 * node, start, mid, l, r)
        rightSum = queryRange(2 * node + 1, mid + 1, end, l, r)
        return leftSum + rightSum
    }
}
```

## Common Operations

| Operation Type | Combine Function | Identity Value |
|----------------|------------------|----------------|
| Sum            | a + b            | 0              |
| Product        | a * b            | 1              |
| Minimum        | min(a, b)        | +∞             |
| Maximum        | max(a, b)        | -∞             |
| GCD            | gcd(a, b)        | 0              |
| LCM            | lcm(a, b)        | 1              |
| AND            | a & b            | ~0 (all 1s)    |
| OR             | a | b            | 0              |
| XOR            | a ^ b            | 0              |

## Use Cases

- **Range sum/min/max queries**: With updates
- **Count elements in range**: With value constraints
- **Finding k-th element**: In a range
- **Interval scheduling**: Resource allocation
- **Computational geometry**: 2D range queries
- **Time series analysis**: Rolling aggregates

## Advantages

- **O(log n) operations**: Both queries and updates
- **Flexible aggregation**: Any associative operation
- **Range updates**: Efficient with lazy propagation
- **Versatile**: Many problem variations
- **Persistent version**: Can track history

## Disadvantages

- **O(n) space**: 4n array allocation
- **Complex implementation**: Especially lazy propagation
- **Static size**: Fixed after construction
- **Not cache-optimal**: Random access pattern
- **2D complexity**: Quadratic space for 2D

## Comparison with Alternatives

| Aspect          | Segment Tree | Fenwick Tree | Sparse Table | Sqrt Decomp |
|-----------------|--------------|--------------|--------------|-------------|
| Build           | O(n)         | O(n log n)   | O(n log n)   | O(n)        |
| Query           | O(log n)     | O(log n)     | O(1)         | O(√n)       |
| Update          | O(log n)     | O(log n)     | O(n log n)   | O(√n)       |
| Range update    | O(log n)*    | O(log n)**   | N/A          | O(√n)       |
| Space           | O(n)         | O(n)         | O(n log n)   | O(n)        |
| Implementation  | Medium       | Simple       | Simple       | Simple      |

*With lazy propagation
**With range Fenwick tree

## Common Pitfalls

- **Array size**: Allocate 4n to be safe
- **Index bounds**: Off-by-one errors common
- **Lazy propagation order**: Push down before recursing
- **Identity value**: Wrong identity breaks queries
- **Combine associativity**: Operation must be associative
- **0 vs 1 indexing**: Be consistent throughout

## Related Structures

- **Fenwick Tree**: Simpler for prefix sums
- **Sparse Table**: O(1) query, no updates
- **Sqrt Decomposition**: Simpler alternative
- **2D Segment Tree**: For 2D range queries
- **Persistent Segment Tree**: Version history
- **Merge Sort Tree**: For order statistics

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
