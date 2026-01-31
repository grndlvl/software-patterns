# van Emde Boas Tree

## Overview

A van Emde Boas (vEB) tree is a tree data structure that implements an associative array with integer keys from a bounded universe. It achieves O(log log u) time for all operations, where u is the universe size. This makes it extremely efficient when the key range is known and bounded, outperforming balanced BSTs for operations on integers from a fixed universe.

The key insight is recursive decomposition: a universe of size u is divided into √u clusters of size √u each, with a summary structure tracking non-empty clusters.

## Properties

- **Universe size u**: Keys must be integers in [0, u-1]
- **Recursive structure**: √u clusters, each a vEB tree of size √u
- **Summary structure**: vEB tree tracking non-empty clusters
- **Minimum/Maximum**: Stored directly for O(1) access
- **Space**: O(u) in basic form, O(n) with hashing

## Time Complexity

| Operation    | Time         | Notes                           |
|--------------|--------------|--------------------------------|
| Member       | O(log log u) | Check if key exists            |
| Insert       | O(log log u) | Add key to set                 |
| Delete       | O(log log u) | Remove key from set            |
| Minimum      | O(1)         | Stored at root                 |
| Maximum      | O(1)         | Stored at root                 |
| Successor    | O(log log u) | Next larger key                |
| Predecessor  | O(log log u) | Next smaller key               |

**Key insight**: T(u) = T(√u) + O(1) solves to O(log log u)

## Space Complexity

- **Basic**: O(u) - full tree for universe
- **With hashing**: O(n log log u) where n is number of stored elements
- **Practical**: Only allocate non-empty subtrees

## Structure

### Universe Size Requirements

For clean recursion, u should be 2^(2^k) for some k:
- u = 2 (base case)
- u = 4, 16, 256, 65536, ...

For arbitrary u, use u = 2^⌈log₂ u⌉ rounded up to next power of 2.

### Index Functions

```pseudocode
// For universe size u:
// high(x) = cluster index = x / √u
// low(x) = position within cluster = x mod √u
// index(i, j) = reconstruct key = i * √u + j

function high(x, u) {
    return floor(x / sqrt(u))
}

function low(x, u) {
    return x mod sqrt(u)
}

function index(i, j, u) {
    return i * sqrt(u) + j
}
```

### Node Structure

```pseudocode
class VEBTree {
    u              // Universe size
    min = null     // Minimum element (stored separately)
    max = null     // Maximum element
    summary = null // vEB tree of size √u (tracks non-empty clusters)
    cluster[]      // Array of √u vEB trees, each of size √u
}

function createVEB(u) {
    veb = new VEBTree()
    veb.u = u
    veb.min = null
    veb.max = null

    if u > 2 {
        sqrtU = ceil(sqrt(u))
        veb.summary = createVEB(sqrtU)
        veb.cluster = new Array(sqrtU)
        for i from 0 to sqrtU - 1 {
            veb.cluster[i] = createVEB(sqrtU)
        }
    }

    return veb
}
```

## Operations

### Minimum and Maximum

```pseudocode
function minimum(veb) {
    return veb.min
}

function maximum(veb) {
    return veb.max
}
```

### Member (Search)

```pseudocode
function member(veb, x) {
    if x == veb.min or x == veb.max {
        return true
    }

    if veb.u == 2 {
        return false  // Base case: not min or max
    }

    // Recurse into appropriate cluster
    return member(veb.cluster[high(x, veb.u)], low(x, veb.u))
}
```

### Successor

```pseudocode
function successor(veb, x) {
    // Base case
    if veb.u == 2 {
        if x == 0 and veb.max == 1 {
            return 1
        }
        return null
    }

    // Case 1: x < min, successor is min
    if veb.min != null and x < veb.min {
        return veb.min
    }

    // Case 2: Check within same cluster
    clusterIndex = high(x, veb.u)
    posInCluster = low(x, veb.u)

    maxInCluster = maximum(veb.cluster[clusterIndex])

    if maxInCluster != null and posInCluster < maxInCluster {
        // Successor is in same cluster
        offset = successor(veb.cluster[clusterIndex], posInCluster)
        return index(clusterIndex, offset, veb.u)
    }

    // Case 3: Find next non-empty cluster
    nextCluster = successor(veb.summary, clusterIndex)

    if nextCluster == null {
        return null  // No successor
    }

    offset = minimum(veb.cluster[nextCluster])
    return index(nextCluster, offset, veb.u)
}
```

### Predecessor

```pseudocode
function predecessor(veb, x) {
    // Base case
    if veb.u == 2 {
        if x == 1 and veb.min == 0 {
            return 0
        }
        return null
    }

    // Case 1: x > max, predecessor is max
    if veb.max != null and x > veb.max {
        return veb.max
    }

    // Case 2: Check within same cluster
    clusterIndex = high(x, veb.u)
    posInCluster = low(x, veb.u)

    minInCluster = minimum(veb.cluster[clusterIndex])

    if minInCluster != null and posInCluster > minInCluster {
        // Predecessor is in same cluster
        offset = predecessor(veb.cluster[clusterIndex], posInCluster)
        return index(clusterIndex, offset, veb.u)
    }

    // Case 3: Find previous non-empty cluster
    prevCluster = predecessor(veb.summary, clusterIndex)

    if prevCluster == null {
        // Check if min is predecessor
        if veb.min != null and x > veb.min {
            return veb.min
        }
        return null
    }

    offset = maximum(veb.cluster[prevCluster])
    return index(prevCluster, offset, veb.u)
}
```

### Insert

```pseudocode
function insert(veb, x) {
    // Empty tree
    if veb.min == null {
        veb.min = x
        veb.max = x
        return
    }

    // x becomes new min, insert old min
    if x < veb.min {
        swap(x, veb.min)
    }

    // Update max
    if x > veb.max {
        veb.max = x
    }

    // Recurse (only if not base case)
    if veb.u > 2 {
        clusterIndex = high(x, veb.u)
        posInCluster = low(x, veb.u)

        if minimum(veb.cluster[clusterIndex]) == null {
            // First element in cluster, update summary
            insert(veb.summary, clusterIndex)
            // Just set min/max, don't recurse further
            veb.cluster[clusterIndex].min = posInCluster
            veb.cluster[clusterIndex].max = posInCluster
        } else {
            // Cluster not empty, recurse into it
            insert(veb.cluster[clusterIndex], posInCluster)
        }
    }
}
```

### Delete

```pseudocode
function delete(veb, x) {
    // Only one element
    if veb.min == veb.max {
        veb.min = null
        veb.max = null
        return
    }

    // Base case with two elements
    if veb.u == 2 {
        if x == 0 {
            veb.min = 1
        } else {
            veb.min = 0
        }
        veb.max = veb.min
        return
    }

    // Deleting min: find new min
    if x == veb.min {
        firstCluster = minimum(veb.summary)
        x = index(firstCluster, minimum(veb.cluster[firstCluster]), veb.u)
        veb.min = x
    }

    // Delete x from its cluster
    clusterIndex = high(x, veb.u)
    posInCluster = low(x, veb.u)

    delete(veb.cluster[clusterIndex], posInCluster)

    // Update summary if cluster became empty
    if minimum(veb.cluster[clusterIndex]) == null {
        delete(veb.summary, clusterIndex)

        // Update max if x was max
        if x == veb.max {
            summaryMax = maximum(veb.summary)
            if summaryMax == null {
                veb.max = veb.min  // Only min remains
            } else {
                veb.max = index(summaryMax, maximum(veb.cluster[summaryMax]), veb.u)
            }
        }
    } else if x == veb.max {
        // Cluster not empty, update max from cluster
        veb.max = index(clusterIndex, maximum(veb.cluster[clusterIndex]), veb.u)
    }
}
```

## Space-Efficient Variant

### Using Hash Tables

```pseudocode
class SpaceEfficientVEB {
    u
    min = null
    max = null
    summary = null
    clusterMap = new HashMap()  // Sparse clusters

    function getOrCreateCluster(index) {
        if index not in this.clusterMap {
            this.clusterMap[index] = new SpaceEfficientVEB(sqrt(u))
        }
        return this.clusterMap[index]
    }
}

// Space: O(n log log u) where n is number of elements
```

## Use Cases

### IP Routing Table

```pseudocode
function buildRoutingTable(prefixes) {
    // IPv4 addresses: u = 2^32
    veb = createVEB(2^32)

    for prefix in prefixes {
        insert(veb, prefix.address)
    }

    return veb
}

function longestPrefixMatch(veb, ip) {
    // Find predecessor for longest matching prefix
    return predecessor(veb, ip)
}
```

### Priority Queue with Integer Priorities

```pseudocode
class IntegerPriorityQueue {
    veb = createVEB(MAX_PRIORITY)
    items = new HashMap()  // priority -> list of items

    function enqueue(item, priority) {
        if priority not in items {
            items[priority] = []
            insert(veb, priority)
        }
        items[priority].append(item)
    }

    function dequeueMin() {
        minPriority = minimum(veb)
        item = items[minPriority].pop()
        if items[minPriority].isEmpty() {
            delete(veb, minPriority)
            delete items[minPriority]
        }
        return item
    }
}
```

### Range Queries

```pseudocode
function rangeQuery(veb, low, high) {
    result = []
    current = successor(veb, low - 1)

    while current != null and current <= high {
        result.append(current)
        current = successor(veb, current)
    }

    return result
}
```

## Advantages

- **Extremely fast**: O(log log u) is nearly constant for practical universes
- **Optimal predecessor/successor**: Faster than balanced BSTs
- **Integer keys**: Direct handling without hashing overhead
- **All operations same complexity**: Uniform performance
- **Cache efficiency**: Predictable access patterns

## Disadvantages

- **Bounded universe**: Keys must be in [0, u-1]
- **Space overhead**: O(u) in basic form
- **Integer keys only**: Not applicable to general keys
- **Implementation complexity**: Recursive structure is intricate
- **Universe size constraints**: Works best with u = 2^(2^k)
- **Not persistent**: Hard to make immutable version

## Comparison with Alternatives

| Operation    | BST (balanced) | Hash Table | vEB Tree    |
|--------------|----------------|------------|-------------|
| Search       | O(log n)       | O(1)*      | O(log log u)|
| Insert       | O(log n)       | O(1)*      | O(log log u)|
| Delete       | O(log n)       | O(1)*      | O(log log u)|
| Successor    | O(log n)       | O(n)       | O(log log u)|
| Predecessor  | O(log n)       | O(n)       | O(log log u)|
| Min/Max      | O(log n)       | O(n)       | O(1)        |
| Range query  | O(log n + k)   | O(n)       | O(log log u + k)|

*Expected, assuming good hash function

## Common Pitfalls

- **Universe size not power of 2**: Need to round up
- **Forgetting min is stored separately**: Min is not in clusters
- **Incorrect high/low calculation**: Use consistent √u
- **Not updating summary**: Must maintain summary on insert/delete
- **Space explosion**: Basic implementation uses O(u) even for few elements
- **Recursion depth**: Deep for large u, may need iterative version
- **Off-by-one in successor/predecessor**: Edge cases at boundaries

## Related Structures

- **X-fast Trie**: O(log log u) using hashing, O(n log u) space
- **Y-fast Trie**: O(log log u) time, O(n) space
- **Fusion Tree**: O(log n / log log n) for predecessor
- **Stratified Tree**: Practical alternative for smaller universes
- **Integer BST**: Simpler but O(log n) operations

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
