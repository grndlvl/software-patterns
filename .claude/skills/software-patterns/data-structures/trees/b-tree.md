# B-Tree

## Overview

A B-tree is a self-balancing tree data structure that maintains sorted data and allows searches, insertions, and deletions in logarithmic time. Unlike binary trees, B-tree nodes can have many children, making them ideal for storage systems that read and write large blocks of data, such as databases and file systems.

## Properties

A B-tree of order m (also called minimum degree t) has these properties:

1. **Root exceptions**: Root may have fewer children (minimum 2 if not leaf)
2. **Node capacity**: Each non-root node has at least t-1 keys and at most 2t-1 keys
3. **Children count**: Non-leaf nodes with k keys have k+1 children
4. **All leaves same depth**: Tree is perfectly balanced
5. **Sorted keys**: Keys within a node are sorted in ascending order

### Terminology

```
Order (m): Maximum number of children per node
Minimum degree (t): Minimum keys = t-1, Maximum keys = 2t-1

Common choices:
- t = 2: 2-3-4 tree (1-3 keys, 2-4 children)
- t = 100: Database index (99-199 keys, 100-200 children)
```

## Time Complexity

| Operation | Average     | Worst       |
|-----------|-------------|-------------|
| Search    | O(log n)    | O(log n)    |
| Insert    | O(log n)    | O(log n)    |
| Delete    | O(log n)    | O(log n)    |
| Min/Max   | O(log n)    | O(log n)    |

Height: O(log_t n) where t is minimum degree

## Space Complexity

O(n) for n keys. Nodes may have wasted space due to minimum fill requirement (typically 50% minimum utilization).

## Operations

### Node Structure

```pseudocode
class BTreeNode {
    keys[]          // Array of keys
    children[]      // Array of child pointers
    n               // Current number of keys
    isLeaf          // True if leaf node
    t               // Minimum degree

    function constructor(t, isLeaf) {
        this.t = t
        this.isLeaf = isLeaf
        this.keys = new Array(2 * t - 1)
        this.children = new Array(2 * t)
        this.n = 0
    }

    function isFull() {
        return this.n == 2 * this.t - 1
    }
}
```

### Search

```pseudocode
function search(node, key) {
    // Find first key greater than or equal to key
    i = 0
    while i < node.n and key > node.keys[i] {
        i = i + 1
    }

    // If found
    if i < node.n and key == node.keys[i] {
        return (node, i)
    }

    // If leaf, key doesn't exist
    if node.isLeaf {
        return null
    }

    // Recurse to appropriate child
    return search(node.children[i], key)
}
```

### Insert

```pseudocode
function insert(tree, key) {
    root = tree.root

    // If root is full, split it
    if root.isFull() {
        newRoot = new BTreeNode(tree.t, false)
        newRoot.children[0] = root
        splitChild(newRoot, 0, root)

        // Decide which child gets new key
        i = 0
        if newRoot.keys[0] < key {
            i = i + 1
        }
        insertNonFull(newRoot.children[i], key)

        tree.root = newRoot
    } else {
        insertNonFull(root, key)
    }
}

function insertNonFull(node, key) {
    i = node.n - 1

    if node.isLeaf {
        // Shift keys and insert
        while i >= 0 and key < node.keys[i] {
            node.keys[i + 1] = node.keys[i]
            i = i - 1
        }
        node.keys[i + 1] = key
        node.n = node.n + 1
    } else {
        // Find child to descend to
        while i >= 0 and key < node.keys[i] {
            i = i - 1
        }
        i = i + 1

        // Split child if full
        if node.children[i].isFull() {
            splitChild(node, i, node.children[i])
            if key > node.keys[i] {
                i = i + 1
            }
        }
        insertNonFull(node.children[i], key)
    }
}

function splitChild(parent, index, child) {
    t = child.t

    // Create new node for right half
    newNode = new BTreeNode(t, child.isLeaf)
    newNode.n = t - 1

    // Copy right half of keys to new node
    for j from 0 to t - 2 {
        newNode.keys[j] = child.keys[j + t]
    }

    // Copy right half of children if not leaf
    if not child.isLeaf {
        for j from 0 to t - 1 {
            newNode.children[j] = child.children[j + t]
        }
    }

    child.n = t - 1

    // Make room in parent for new child
    for j from parent.n down to index + 1 {
        parent.children[j + 1] = parent.children[j]
    }
    parent.children[index + 1] = newNode

    // Move middle key up to parent
    for j from parent.n - 1 down to index {
        parent.keys[j + 1] = parent.keys[j]
    }
    parent.keys[index] = child.keys[t - 1]
    parent.n = parent.n + 1
}
```

### Delete

```pseudocode
function delete(tree, key) {
    deleteFromNode(tree.root, key)

    // If root has no keys, make first child new root
    if tree.root.n == 0 {
        if tree.root.isLeaf {
            tree.root = null
        } else {
            tree.root = tree.root.children[0]
        }
    }
}

function deleteFromNode(node, key) {
    i = findKeyIndex(node, key)

    // Case 1: Key is in this node and node is leaf
    if i < node.n and node.keys[i] == key {
        if node.isLeaf {
            removeFromLeaf(node, i)
        } else {
            removeFromNonLeaf(node, i)
        }
    } else {
        // Key not in this node
        if node.isLeaf {
            return  // Key doesn't exist
        }

        // Ensure child has enough keys
        flag = (i == node.n)
        if node.children[i].n < node.t {
            fill(node, i)
        }

        if flag and i > node.n {
            deleteFromNode(node.children[i - 1], key)
        } else {
            deleteFromNode(node.children[i], key)
        }
    }
}

function removeFromLeaf(node, index) {
    for i from index + 1 to node.n - 1 {
        node.keys[i - 1] = node.keys[i]
    }
    node.n = node.n - 1
}

function removeFromNonLeaf(node, index) {
    key = node.keys[index]

    // Case 2a: Left child has >= t keys
    if node.children[index].n >= node.t {
        predecessor = getPredecessor(node, index)
        node.keys[index] = predecessor
        deleteFromNode(node.children[index], predecessor)
    }
    // Case 2b: Right child has >= t keys
    else if node.children[index + 1].n >= node.t {
        successor = getSuccessor(node, index)
        node.keys[index] = successor
        deleteFromNode(node.children[index + 1], successor)
    }
    // Case 2c: Both children have t-1 keys - merge
    else {
        merge(node, index)
        deleteFromNode(node.children[index], key)
    }
}

function fill(node, index) {
    // Borrow from left sibling
    if index != 0 and node.children[index - 1].n >= node.t {
        borrowFromPrev(node, index)
    }
    // Borrow from right sibling
    else if index != node.n and node.children[index + 1].n >= node.t {
        borrowFromNext(node, index)
    }
    // Merge with sibling
    else {
        if index != node.n {
            merge(node, index)
        } else {
            merge(node, index - 1)
        }
    }
}

function merge(node, index) {
    child = node.children[index]
    sibling = node.children[index + 1]
    t = child.t

    // Move key from parent to child
    child.keys[t - 1] = node.keys[index]

    // Copy keys from sibling
    for i from 0 to sibling.n - 1 {
        child.keys[i + t] = sibling.keys[i]
    }

    // Copy children from sibling
    if not child.isLeaf {
        for i from 0 to sibling.n {
            child.children[i + t] = sibling.children[i]
        }
    }

    // Shift keys in parent
    for i from index + 1 to node.n - 1 {
        node.keys[i - 1] = node.keys[i]
    }

    // Shift children in parent
    for i from index + 2 to node.n {
        node.children[i - 1] = node.children[i]
    }

    child.n = child.n + sibling.n + 1
    node.n = node.n - 1
}
```

## Implementation

```pseudocode
class BTree {
    root = null
    t               // Minimum degree

    function constructor(t) {
        this.t = t
        this.root = new BTreeNode(t, true)
    }

    function search(key) {
        if this.root == null {
            return null
        }
        return searchInNode(this.root, key)
    }

    function searchInNode(node, key) {
        i = 0
        while i < node.n and key > node.keys[i] {
            i = i + 1
        }

        if i < node.n and key == node.keys[i] {
            return (node, i)
        }

        if node.isLeaf {
            return null
        }

        return searchInNode(node.children[i], key)
    }

    function insert(key) {
        // Implementation from above
    }

    function delete(key) {
        // Implementation from above
    }

    function traverse() {
        if this.root != null {
            traverseNode(this.root)
        }
    }

    function traverseNode(node) {
        for i from 0 to node.n - 1 {
            if not node.isLeaf {
                traverseNode(node.children[i])
            }
            process(node.keys[i])
        }

        if not node.isLeaf {
            traverseNode(node.children[node.n])
        }
    }
}
```

## Variants

### B+ Tree

- All keys stored in leaves
- Internal nodes only have keys for routing
- Leaves linked for range queries
- Most common in databases

### B* Tree

- Minimum 2/3 full (vs 1/2 for B-tree)
- Delays splits by redistributing to siblings
- Better space utilization

## Use Cases

- **Database systems**: Primary index structure (MySQL InnoDB, PostgreSQL)
- **File systems**: Directory indexing (NTFS, ext4, HFS+)
- **Key-value stores**: LevelDB, RocksDB
- **Search engines**: Inverted index storage
- **Operating systems**: Virtual memory management
- **Any disk-based storage**: Optimized for block I/O

## Advantages

- **Optimal for disk I/O**: Minimizes disk reads with high branching factor
- **Guaranteed O(log n)**: Perfectly balanced
- **High fanout**: Shallow tree even for billions of keys
- **Sequential access**: Good for range queries (especially B+ trees)
- **Efficient updates**: Localized changes, no full rebalancing

## Disadvantages

- **Complex implementation**: Many edge cases
- **Space overhead**: Nodes may be half empty
- **In-memory overhead**: Less cache-efficient than binary trees
- **Insert/delete complexity**: Splits and merges
- **Not ideal for RAM**: Binary trees often better for in-memory use

## Comparison with Alternatives

| Aspect          | B-Tree      | Red-Black   | Hash Table  | LSM Tree    |
|-----------------|-------------|-------------|-------------|-------------|
| Search          | O(log n)    | O(log n)    | O(1)        | O(log n)    |
| Insert          | O(log n)    | O(log n)    | O(1)        | O(log n)*   |
| Range query     | Excellent   | Good        | Poor        | Good        |
| Disk I/O        | Optimal     | Poor        | Random      | Sequential  |
| Cache behavior  | Block-based | Poor        | Random      | Sequential  |
| Best for        | Databases   | In-memory   | Key lookup  | Write-heavy |

*Amortized

## Common Pitfalls

- **Order vs degree**: Different definitions in literature
- **Underflow handling**: Must borrow or merge correctly
- **Root special cases**: Different minimum for root
- **Off-by-one**: In split and merge operations
- **Memory management**: Freeing merged/split nodes
- **Concurrent access**: Requires careful locking strategy

## Related Structures

- **B+ Tree**: Keys only in leaves, linked leaves
- **B* Tree**: Higher minimum fill factor
- **2-3 Tree**: B-tree with t=2
- **2-3-4 Tree**: B-tree with t=2, maps to Red-Black
- **Bε-tree**: Optimized for write amplification
- **LSM Tree**: Log-structured, write-optimized

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
