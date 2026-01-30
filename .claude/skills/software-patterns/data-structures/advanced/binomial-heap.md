# Binomial Heap

## Overview

A Binomial Heap is a collection of binomial trees that satisfies the min-heap property. It supports efficient merge operations in O(log n) time, making it suitable for priority queue implementations where merging heaps is frequent. Binomial heaps serve as the foundation for understanding Fibonacci heaps and provide a good balance between implementation complexity and performance guarantees.

A binomial tree Bk is defined recursively:
- B0 is a single node
- Bk consists of two Bk-1 trees, with one tree's root becoming a child of the other's root

## Properties

- **Binomial tree Bk**: Has exactly 2^k nodes and height k
- **Structure**: Collection of binomial trees with distinct orders
- **Min-heap ordered**: Parent key ≤ child keys in each tree
- **Unique orders**: At most one tree of each order (like binary representation)
- **n nodes**: At most ⌊log₂(n)⌋ + 1 binomial trees

### Binomial Tree Properties

| Property | Value for Bk |
|----------|-------------|
| Nodes | 2^k |
| Height | k |
| Root degree | k |
| Nodes at depth d | C(k, d) |

## Time Complexity

| Operation    | Time      | Notes                           |
|--------------|-----------|--------------------------------|
| Make-Heap    | O(1)      | Create empty heap              |
| Find-Min     | O(log n)  | Scan root list (O(1) with pointer) |
| Insert       | O(log n)  | Create B0 and merge            |
| Union/Merge  | O(log n)  | Like binary addition           |
| Extract-Min  | O(log n)  | Remove root, merge children    |
| Decrease-Key | O(log n)  | Bubble up in tree              |
| Delete       | O(log n)  | Decrease-key + extract-min     |

## Space Complexity

O(n) for n elements. Each node stores key, degree, parent, child, and sibling pointers.

## Operations

### Node Structure

```pseudocode
class BinomialNode {
    key              // Element value
    degree = 0       // Number of children
    parent = null    // Parent pointer
    child = null     // Leftmost child
    sibling = null   // Right sibling in root list
}

class BinomialHeap {
    head = null      // Head of root list
}
```

### Make-Heap

```pseudocode
function makeHeap() {
    heap = new BinomialHeap()
    heap.head = null
    return heap
}
```

### Find-Min

```pseudocode
function findMin(heap) {
    if heap.head == null {
        throw HeapEmptyError
    }

    min = infinity
    minNode = null
    current = heap.head

    while current != null {
        if current.key < min {
            min = current.key
            minNode = current
        }
        current = current.sibling
    }

    return minNode.key
}
```

### Link Two Binomial Trees

```pseudocode
function link(y, z) {
    // Make y a child of z (both are Bk, result is Bk+1)
    // Assumes z.key <= y.key
    y.parent = z
    y.sibling = z.child
    z.child = y
    z.degree = z.degree + 1
}
```

### Merge Root Lists

```pseudocode
function mergeRootLists(h1, h2) {
    // Merge two root lists sorted by degree
    if h1 == null {
        return h2
    }
    if h2 == null {
        return h1
    }

    head = null
    tail = null

    while h1 != null and h2 != null {
        if h1.degree <= h2.degree {
            next = h1
            h1 = h1.sibling
        } else {
            next = h2
            h2 = h2.sibling
        }

        if head == null {
            head = next
            tail = next
        } else {
            tail.sibling = next
            tail = next
        }
    }

    // Append remaining
    if h1 != null {
        tail.sibling = h1
    } else if h2 != null {
        tail.sibling = h2
    }

    return head
}
```

### Union (Merge Two Heaps)

```pseudocode
function union(heap1, heap2) {
    newHeap = makeHeap()
    newHeap.head = mergeRootLists(heap1.head, heap2.head)

    if newHeap.head == null {
        return newHeap
    }

    prev = null
    curr = newHeap.head
    next = curr.sibling

    while next != null {
        // Case 1 & 2: Degrees differ or three trees of same degree
        if curr.degree != next.degree or
           (next.sibling != null and next.sibling.degree == curr.degree) {
            prev = curr
            curr = next
        }
        // Case 3: curr.key <= next.key, link next under curr
        else if curr.key <= next.key {
            curr.sibling = next.sibling
            link(next, curr)
        }
        // Case 4: curr.key > next.key, link curr under next
        else {
            if prev == null {
                newHeap.head = next
            } else {
                prev.sibling = next
            }
            link(curr, next)
            curr = next
        }
        next = curr.sibling
    }

    return newHeap
}
```

### Insert

```pseudocode
function insert(heap, key) {
    newHeap = makeHeap()
    node = new BinomialNode()
    node.key = key
    newHeap.head = node

    return union(heap, newHeap)
}
```

### Extract-Min

```pseudocode
function extractMin(heap) {
    if heap.head == null {
        throw HeapEmptyError
    }

    // Find minimum root and its predecessor
    minPrev = null
    min = heap.head
    prev = null
    curr = heap.head

    while curr != null {
        if curr.key < min.key {
            min = curr
            minPrev = prev
        }
        prev = curr
        curr = curr.sibling
    }

    // Remove min from root list
    if minPrev == null {
        heap.head = min.sibling
    } else {
        minPrev.sibling = min.sibling
    }

    // Reverse min's children to form new heap
    childHeap = makeHeap()
    child = min.child
    while child != null {
        next = child.sibling
        child.sibling = childHeap.head
        child.parent = null
        childHeap.head = child
        child = next
    }

    // Union original heap with children
    heap = union(heap, childHeap)

    return min.key
}
```

### Decrease-Key

```pseudocode
function decreaseKey(heap, node, newKey) {
    if newKey > node.key {
        throw InvalidKeyError
    }

    node.key = newKey

    // Bubble up
    curr = node
    parent = curr.parent

    while parent != null and curr.key < parent.key {
        // Swap keys (or swap nodes)
        swap(curr.key, parent.key)
        curr = parent
        parent = curr.parent
    }
}
```

### Delete

```pseudocode
function delete(heap, node) {
    decreaseKey(heap, node, -infinity)
    extractMin(heap)
}
```

## Binary Representation Analogy

The structure of a binomial heap mirrors binary number representation:

```
n = 13 = 1101₂ = 2³ + 2² + 2⁰

Binomial heap with 13 nodes has:
- One B3 (8 nodes)
- One B2 (4 nodes)
- One B0 (1 node)

Insert is like binary increment:
13 + 1 = 14 = 1110₂
- B0 + new node → B1 (carry)
- B1 + carry → B2 (carry)
- B2 + carry → B3 (carry)
- B3 + carry → stored as B3, carry B4
Result: B3, B2, B1 (no B0)
```

## Use Cases

### Priority Queue with Frequent Merges

```pseudocode
function processJobQueues(queues) {
    // Merge all job queues efficiently
    combined = makeHeap()

    for queue in queues {
        combined = union(combined, queue)
    }

    // Process jobs by priority
    while combined.head != null {
        job = extractMin(combined)
        process(job)
    }
}
```

### Distributed Systems

```pseudocode
function mergePartitionResults(partitions) {
    // Each partition has its own heap of results
    // Merge them efficiently for final processing

    result = makeHeap()
    for partition in partitions {
        result = union(result, partition.resultHeap)
    }

    return result
}
```

## Advantages

- **Efficient merge**: O(log n) union operation
- **Predictable structure**: Based on binary representation
- **Worst-case bounds**: All operations are O(log n) worst-case
- **Foundation for Fibonacci heaps**: Similar structure, simpler
- **Good for batch operations**: Multiple insertions can be merged

## Disadvantages

- **No O(1) decrease-key**: Unlike Fibonacci heaps
- **More pointers than binary heap**: Parent, child, sibling
- **Find-min is O(log n)**: Unless maintaining min pointer
- **Complex implementation**: More intricate than binary heap
- **Cache unfriendly**: Pointer-based structure

## Comparison with Alternatives

| Aspect           | Binary Heap | Binomial Heap | Fibonacci Heap |
|------------------|-------------|---------------|----------------|
| Find-Min         | O(1)        | O(log n)*     | O(1)           |
| Insert           | O(log n)    | O(log n)      | O(1)*          |
| Extract-Min      | O(log n)    | O(log n)      | O(log n)*      |
| Decrease-Key     | O(log n)    | O(log n)      | O(1)*          |
| Union            | O(n)        | O(log n)      | O(1)           |
| Implementation   | Simple      | Moderate      | Complex        |
| Space per node   | 1 value     | 4 pointers    | 5 pointers     |

*O(1) with min pointer, *Amortized

## Common Pitfalls

- **Forgetting to reverse children**: In extract-min, children must be reversed
- **Incorrect linking order**: Always link larger key under smaller
- **Missing case in union**: Four cases for combining trees
- **Not handling empty heaps**: Check for null in merge operations
- **Sibling pointer management**: Easy to create cycles or lose nodes
- **Degree update after link**: Must increment parent's degree

## Related Structures

- **Fibonacci Heap**: Lazy version with O(1) amortized operations
- **Pairing Heap**: Simpler with good practical performance
- **Leftist Heap**: Another mergeable heap variant
- **Skew Heap**: Self-adjusting mergeable heap
- **Brodal Queue**: Worst-case optimal heap

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
