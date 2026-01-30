# Fibonacci Heap

## Overview

A Fibonacci heap is an advanced heap data structure consisting of a collection of min-heap-ordered trees. It provides better amortized time bounds than binary heaps for several operations, particularly decrease-key and merge. Fibonacci heaps achieve these improvements through lazy consolidation - deferring structural work until absolutely necessary. They are theoretically optimal for graph algorithms like Dijkstra's shortest path and Prim's minimum spanning tree, though their practical performance benefits appear primarily in dense graphs with many decrease-key operations.

The name "Fibonacci heap" comes from the Fibonacci numbers used in the runtime analysis, which prove that trees within the heap have a logarithmic height bound.

## Properties

- **Min-heap ordered**: Each node's key is greater than or equal to its parent's key
- **Collection of trees**: Multiple heap-ordered trees, not a single tree
- **Lazy operations**: Structural consolidation deferred until extract-min
- **Circular doubly-linked root list**: Roots stored in circular list for O(1) concatenation
- **Child lists**: Each node's children in circular doubly-linked lists
- **Marked nodes**: Tracks nodes that have lost a child (for cascading cuts)
- **Minimum pointer**: Direct pointer to minimum element for O(1) access
- **Degree tracking**: Each node stores its degree (number of children)
- **Potential function**: Amortized analysis uses potential based on trees and marked nodes

## Time Complexity

| Operation      | Amortized   | Worst Case  | Notes                           |
|----------------|-------------|-------------|---------------------------------|
| Find-Min       | O(1)        | O(1)        | Direct pointer to minimum       |
| Insert         | O(1)        | O(1)        | Add as new root, update min     |
| Union (Merge)  | O(1)        | O(1)        | Concatenate root lists          |
| Decrease-Key   | O(1)        | O(n)        | Cut and cascading cut           |
| Extract-Min    | O(log n)    | O(n)        | Consolidate trees               |
| Delete         | O(log n)    | O(n)        | Decrease-key + extract-min      |

**Key Insight**: The O(1) amortized decrease-key is what makes Fibonacci heaps asymptotically optimal for Dijkstra's algorithm, reducing it from O((V + E) log V) with binary heaps to O(E + V log V) with Fibonacci heaps.

## Space Complexity

O(n) for n elements. Each node requires additional storage for:
- Parent, child, left, and right pointers
- Degree counter
- Mark flag

Constant factor overhead is higher than binary heaps.

## Operations

### Node Structure

```pseudocode
class FibNode {
    key              // Element value
    degree = 0       // Number of children
    parent = null    // Parent pointer
    child = null     // One child pointer (others via circular list)
    left = this      // Left sibling in circular list
    right = this     // Right sibling in circular list
    mark = false     // Whether node has lost a child since becoming child
}
```

### Insert

Add element as new tree in root list:

```pseudocode
function insert(heap, key) {
    // Create new node
    node = new FibNode()
    node.key = key
    node.degree = 0
    node.mark = false
    node.parent = null
    node.child = null
    node.left = node
    node.right = node

    // Add to root list
    if heap.min == null {
        heap.min = node
    } else {
        // Insert into circular root list
        node.right = heap.min.right
        node.left = heap.min
        heap.min.right.left = node
        heap.min.right = node

        // Update minimum if necessary
        if node.key < heap.min.key {
            heap.min = node
        }
    }

    heap.n = heap.n + 1
    return node
}
```

### Find-Min

```pseudocode
function findMin(heap) {
    if heap.min == null {
        throw HeapEmptyError
    }
    return heap.min.key
}
```

### Union (Merge)

Concatenate root lists of two heaps:

```pseudocode
function union(heap1, heap2) {
    if heap1.min == null {
        return heap2
    }
    if heap2.min == null {
        return heap1
    }

    // Concatenate circular root lists
    heap1.min.right.left = heap2.min.left
    heap2.min.left.right = heap1.min.right
    heap1.min.right = heap2.min
    heap2.min.left = heap1.min

    // Update minimum
    if heap2.min.key < heap1.min.key {
        heap1.min = heap2.min
    }

    heap1.n = heap1.n + heap2.n
    return heap1
}
```

### Extract-Min

Remove minimum and consolidate trees:

```pseudocode
function extractMin(heap) {
    z = heap.min

    if z == null {
        throw HeapEmptyError
    }

    // Add all children of min to root list
    if z.child != null {
        child = z.child
        do {
            next = child.right
            // Remove parent pointer
            child.parent = null
            // Add to root list
            child.right = heap.min.right
            child.left = heap.min
            heap.min.right.left = child
            heap.min.right = child
            child = next
        } while child != z.child
    }

    // Remove z from root list
    z.left.right = z.right
    z.right.left = z.left

    if z == z.right {
        // Heap had only one node
        heap.min = null
    } else {
        heap.min = z.right
        consolidate(heap)
    }

    heap.n = heap.n - 1
    return z.key
}

function consolidate(heap) {
    // Array to track trees by degree
    maxDegree = floor(log(heap.n)) + 1
    A = array[0..maxDegree] initialized to null

    // Collect all roots
    roots = []
    current = heap.min
    do {
        roots.append(current)
        current = current.right
    } while current != heap.min

    // Consolidate trees of same degree
    for w in roots {
        x = w
        d = x.degree

        while A[d] != null {
            y = A[d]  // Another tree with same degree

            // Ensure x has smaller key
            if x.key > y.key {
                swap(x, y)
            }

            // Make y a child of x
            link(heap, y, x)
            A[d] = null
            d = d + 1
        }

        A[d] = x
    }

    // Rebuild root list and find new minimum
    heap.min = null
    for i from 0 to maxDegree {
        if A[i] != null {
            if heap.min == null {
                // Create new root list
                A[i].left = A[i]
                A[i].right = A[i]
                heap.min = A[i]
            } else {
                // Add to root list
                A[i].right = heap.min.right
                A[i].left = heap.min
                heap.min.right.left = A[i]
                heap.min.right = A[i]

                if A[i].key < heap.min.key {
                    heap.min = A[i]
                }
            }
        }
    }
}

function link(heap, y, x) {
    // Remove y from root list
    y.left.right = y.right
    y.right.left = y.left

    // Make y a child of x
    y.parent = x

    if x.child == null {
        x.child = y
        y.left = y
        y.right = y
    } else {
        y.right = x.child.right
        y.left = x.child
        x.child.right.left = y
        x.child.right = y
    }

    x.degree = x.degree + 1
    y.mark = false
}
```

### Decrease-Key

Decrease key and restore heap property:

```pseudocode
function decreaseKey(heap, node, newKey) {
    if newKey > node.key {
        throw InvalidKeyError
    }

    node.key = newKey
    parent = node.parent

    if parent != null and node.key < parent.key {
        cut(heap, node, parent)
        cascadingCut(heap, parent)
    }

    if node.key < heap.min.key {
        heap.min = node
    }
}

function cut(heap, x, y) {
    // Remove x from child list of y
    if x.right == x {
        y.child = null
    } else {
        x.left.right = x.right
        x.right.left = x.left
        if y.child == x {
            y.child = x.right
        }
    }

    y.degree = y.degree - 1

    // Add x to root list
    x.right = heap.min.right
    x.left = heap.min
    heap.min.right.left = x
    heap.min.right = x

    x.parent = null
    x.mark = false
}

function cascadingCut(heap, y) {
    z = y.parent

    if z != null {
        if y.mark == false {
            // First child lost, mark it
            y.mark = true
        } else {
            // Second child lost, cut and continue up
            cut(heap, y, z)
            cascadingCut(heap, z)
        }
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

## Use Cases

### Dijkstra's Shortest Path Algorithm

Fibonacci heaps provide asymptotically optimal performance for Dijkstra's algorithm:

```pseudocode
function dijkstra(graph, source) {
    dist = array[graph.V] initialized to infinity
    nodes = array[graph.V]  // Track heap nodes
    heap = new FibonacciHeap()

    dist[source] = 0
    for v in graph.vertices {
        nodes[v] = heap.insert((dist[v], v))
    }

    while not heap.isEmpty() {
        (d, u) = heap.extractMin()

        for (v, weight) in graph.adjacentEdges(u) {
            if dist[u] + weight < dist[v] {
                dist[v] = dist[u] + weight
                heap.decreaseKey(nodes[v], (dist[v], v))
            }
        }
    }

    return dist
}
```

**Time Complexity**: O(E + V log V) vs O((V + E) log V) with binary heap

### Prim's Minimum Spanning Tree

```pseudocode
function prim(graph) {
    key = array[graph.V] initialized to infinity
    parent = array[graph.V] initialized to null
    nodes = array[graph.V]
    inMST = set()
    heap = new FibonacciHeap()

    key[0] = 0
    for v in graph.vertices {
        nodes[v] = heap.insert((key[v], v))
    }

    while not heap.isEmpty() {
        (k, u) = heap.extractMin()
        inMST.add(u)

        for (v, weight) in graph.adjacentEdges(u) {
            if v not in inMST and weight < key[v] {
                key[v] = weight
                parent[v] = u
                heap.decreaseKey(nodes[v], (weight, v))
            }
        }
    }

    return parent
}
```

**Time Complexity**: O(E + V log V) vs O((V + E) log V) with binary heap

### Other Applications

- **Network optimization**: Flow algorithms with capacity updates
- **Job scheduling**: Dynamic priority updates in real-time systems
- **Data structures**: As building block for advanced structures
- **Computational geometry**: Voronoi diagrams, triangulation

## Advantages

- **Asymptotically optimal**: Best theoretical bounds for many operations
- **O(1) decrease-key**: Amortized constant time for priority updates
- **O(1) merge**: Efficient heap union operation
- **O(1) insert**: Constant time insertion
- **Lazy evaluation**: Defers expensive work until necessary
- **Theoretically superior**: For algorithms with many decrease-key operations

## Disadvantages

- **High constant factors**: Practical performance often slower than binary heaps
- **Complex implementation**: Intricate pointer manipulation and edge cases
- **Space overhead**: Additional pointers and metadata per node
- **Cache unfriendly**: Poor spatial locality compared to array-based heaps
- **Amortized bounds**: Worst-case performance can be poor
- **Practical limitations**: Benefits mainly visible in dense graphs
- **Maintenance burden**: Complex code is harder to debug and maintain

## Comparison with Alternatives

| Aspect           | Binary Heap | Binomial Heap | Fibonacci Heap | Pairing Heap   |
|------------------|-------------|---------------|----------------|----------------|
| Find-Min         | O(1)        | O(1)          | O(1)           | O(1)           |
| Insert           | O(log n)    | O(log n)      | O(1)*          | O(1)           |
| Extract-Min      | O(log n)    | O(log n)      | O(log n)*      | O(log n)*      |
| Decrease-Key     | O(log n)    | O(log n)      | O(1)*          | O(log n)*†     |
| Merge/Union      | O(n)        | O(log n)      | O(1)           | O(1)           |
| Delete           | O(log n)    | O(log n)      | O(log n)*      | O(log n)*      |
| Space per node   | 1 value     | 4 pointers    | 5 pointers     | 3 pointers     |
| Implementation   | Simple      | Moderate      | Complex        | Simple         |
| Practical use    | Common      | Rare          | Research       | Uncommon       |
| Cache locality   | Excellent   | Poor          | Poor           | Poor           |

*Amortized
†Conjectured O(log log n) amortized

## Common Pitfalls

- **Premature optimization**: Using Fibonacci heaps when binary heaps suffice
- **Circular list bugs**: Easy to create infinite loops or break circularity
- **Mark bit mismanagement**: Forgetting to reset marks during cuts
- **Parent pointer errors**: Leaving stale parent pointers after operations
- **Consolidate correctness**: Off-by-one errors in degree array sizing
- **Memory leaks**: Circular structures complicate garbage collection
- **Cascading cut depth**: Misunderstanding when cascading stops
- **Degree tracking**: Forgetting to update degree counters
- **Minimum pointer staleness**: Not updating min after modifications
- **Comparison overhead**: Complex operations dominate simple workloads

## Related Structures

- **Binomial Heap**: Simpler mergeable heap with O(log n) operations
- **Pairing Heap**: Simpler alternative with competitive practical performance
- **Strict Fibonacci Heap**: Variant with better worst-case bounds
- **Relaxed Heap**: Allows temporary heap property violations
- **Brodal Queue**: Worst-case optimal (not just amortized) but impractical
- **Rank-Pairing Heap**: Recent alternative with similar bounds
- **Hollow Heap**: Simplified Fibonacci heap variant (2015)
- **Soft Heap**: Approximate heap allowing controlled errors

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
