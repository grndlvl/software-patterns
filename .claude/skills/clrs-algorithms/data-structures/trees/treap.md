# Treap

## Overview

A Treap (Tree + Heap) is a randomized binary search tree that combines BST ordering by keys with heap ordering by random priorities. Each node has a key (following BST property) and a randomly assigned priority (following heap property). The random priorities ensure the tree remains balanced with high probability, giving O(log n) expected time for all operations.

Treaps were introduced by Aragon and Seidel in 1989. They provide a simpler alternative to deterministic balanced trees like AVL and Red-Black trees.

## Properties

- **BST property**: Left subtree keys < node key < right subtree keys
- **Heap property**: Parent priority > child priorities (max-heap)
- **Unique structure**: For a given set of (key, priority) pairs, the treap structure is unique
- **Randomized balance**: Random priorities keep tree balanced in expectation
- **Expected height**: O(log n) with high probability

## Time Complexity

| Operation | Expected  | Worst Case | Notes                        |
|-----------|-----------|------------|------------------------------|
| Search    | O(log n)  | O(n)       | Standard BST search          |
| Insert    | O(log n)  | O(n)       | Insert + rotate up           |
| Delete    | O(log n)  | O(n)       | Rotate down + remove         |
| Split     | O(log n)  | O(n)       | Split at key                 |
| Merge     | O(log n)  | O(n)       | Merge two treaps             |

All bounds are expected; worst case O(n) extremely unlikely.

## Space Complexity

O(n) for n elements. Each node stores key, priority, and two child pointers.

## Structure

### Node Structure

```pseudocode
class TreapNode {
    key
    priority        // Random value
    value           // Optional associated data
    left = null
    right = null
}

class Treap {
    root = null
}

// Priority: higher = closer to root (max-heap)
// Key: standard BST ordering (left < parent < right)
```

## Operations

### Search

Standard BST search (priority doesn't affect search):

```pseudocode
function search(tree, key) {
    node = tree.root

    while node != null {
        if key == node.key {
            return node.value
        } else if key < node.key {
            node = node.left
        } else {
            node = node.right
        }
    }

    return null  // Not found
}
```

### Rotations

```pseudocode
function rotateRight(tree, y) {
    x = y.left
    y.left = x.right
    x.right = y

    // Update root if needed
    return x  // New root of subtree
}

function rotateLeft(tree, x) {
    y = x.right
    x.right = y.left
    y.left = x

    return y  // New root of subtree
}
```

### Insert

Insert as in BST, then rotate up to restore heap property:

```pseudocode
function insert(tree, key, value = null) {
    priority = random()  // Random priority

    tree.root = insertHelper(tree.root, key, value, priority)
}

function insertHelper(node, key, value, priority) {
    if node == null {
        return new TreapNode(key, priority, value)
    }

    if key < node.key {
        node.left = insertHelper(node.left, key, value, priority)

        // Rotate right if heap property violated
        if node.left.priority > node.priority {
            node = rotateRight(tree, node)
        }
    } else if key > node.key {
        node.right = insertHelper(node.right, key, value, priority)

        // Rotate left if heap property violated
        if node.right.priority > node.priority {
            node = rotateLeft(tree, node)
        }
    } else {
        // Key exists, update value
        node.value = value
    }

    return node
}
```

### Delete

Rotate node down until it's a leaf, then remove:

```pseudocode
function delete(tree, key) {
    tree.root = deleteHelper(tree.root, key)
}

function deleteHelper(node, key) {
    if node == null {
        return null  // Key not found
    }

    if key < node.key {
        node.left = deleteHelper(node.left, key)
    } else if key > node.key {
        node.right = deleteHelper(node.right, key)
    } else {
        // Found the node to delete
        if node.left == null {
            return node.right
        } else if node.right == null {
            return node.left
        } else {
            // Rotate down based on child priorities
            if node.left.priority > node.right.priority {
                node = rotateRight(tree, node)
                node.right = deleteHelper(node.right, key)
            } else {
                node = rotateLeft(tree, node)
                node.left = deleteHelper(node.left, key)
            }
        }
    }

    return node
}
```

### Split

Split treap into two treaps at a key:

```pseudocode
function split(tree, key) {
    // Returns (leftTree, rightTree) where:
    // leftTree has all keys < key
    // rightTree has all keys >= key

    return splitHelper(tree.root, key)
}

function splitHelper(node, key) {
    if node == null {
        return (null, null)
    }

    if key <= node.key {
        // Split left subtree
        (left, right) = splitHelper(node.left, key)
        node.left = right
        return (left, node)
    } else {
        // Split right subtree
        (left, right) = splitHelper(node.right, key)
        node.right = left
        return (node, right)
    }
}
```

### Merge

Merge two treaps where all keys in left < all keys in right:

```pseudocode
function merge(leftTree, rightTree) {
    return mergeHelper(leftTree.root, rightTree.root)
}

function mergeHelper(left, right) {
    if left == null {
        return right
    }
    if right == null {
        return left
    }

    // Higher priority becomes root
    if left.priority > right.priority {
        left.right = mergeHelper(left.right, right)
        return left
    } else {
        right.left = mergeHelper(left, right.left)
        return right
    }
}
```

### Alternative Insert/Delete Using Split/Merge

```pseudocode
function insertUsingSplit(tree, key, value) {
    (left, right) = split(tree, key)

    newNode = new TreapNode(key, random(), value)

    tree.root = merge(merge(left, newNode), right)
}

function deleteUsingSplit(tree, key) {
    (left, mid) = split(tree, key)
    (_, right) = split(mid, key + 1)  // Assuming integer keys

    tree.root = merge(left, right)
}
```

## Implicit Treap (Indexed Treap)

Use position as implicit key for array-like operations:

```pseudocode
class ImplicitTreapNode {
    value
    priority
    size = 1        // Subtree size
    left = null
    right = null
}

function getSize(node) {
    return node == null ? 0 : node.size
}

function updateSize(node) {
    if node != null {
        node.size = getSize(node.left) + getSize(node.right) + 1
    }
}

// Access by index
function getByIndex(node, index) {
    leftSize = getSize(node.left)

    if index < leftSize {
        return getByIndex(node.left, index)
    } else if index == leftSize {
        return node.value
    } else {
        return getByIndex(node.right, index - leftSize - 1)
    }
}

// Insert at index
function insertAtIndex(node, index, value) {
    (left, right) = splitBySize(node, index)
    newNode = new ImplicitTreapNode(value, random())
    return merge(merge(left, newNode), right)
}

// Split by size
function splitBySize(node, k) {
    if node == null {
        return (null, null)
    }

    leftSize = getSize(node.left)

    if k <= leftSize {
        (left, right) = splitBySize(node.left, k)
        node.left = right
        updateSize(node)
        return (left, node)
    } else {
        (left, right) = splitBySize(node.right, k - leftSize - 1)
        node.right = left
        updateSize(node)
        return (node, right)
    }
}
```

## Use Cases

### Dynamic Array with Fast Insert/Delete

```pseudocode
class DynamicArray {
    treap = new ImplicitTreap()

    function get(index) {
        return getByIndex(treap.root, index)
    }

    function insert(index, value) {
        treap.root = insertAtIndex(treap.root, index, value)
    }

    function remove(index) {
        treap.root = removeAtIndex(treap.root, index)
    }

    function size() {
        return getSize(treap.root)
    }
}
```

### Range Operations

```pseudocode
class RangeTreap {
    root = null

    // Add lazy propagation for range updates
    function rangeQuery(left, right) {
        (before, rest) = split(root, left)
        (middle, after) = split(rest, right - left + 1)

        result = computeAggregate(middle)

        root = merge(merge(before, middle), after)
        return result
    }

    function rangeUpdate(left, right, delta) {
        (before, rest) = split(root, left)
        (middle, after) = split(rest, right - left + 1)

        applyLazy(middle, delta)

        root = merge(merge(before, middle), after)
    }
}
```

### Rope Data Structure

```pseudocode
class Rope {
    // Efficient string with fast insert, delete, concat
    treap = new ImplicitTreap()

    function charAt(index) {
        return getByIndex(treap.root, index)
    }

    function insert(index, string) {
        for (i, char) in enumerate(string) {
            insertAtIndex(treap.root, index + i, char)
        }
    }

    function concat(other) {
        treap.root = merge(treap.root, other.treap.root)
    }

    function substring(start, end) {
        (before, rest) = splitBySize(treap.root, start)
        (middle, after) = splitBySize(rest, end - start)

        // Collect characters from middle
        result = collectInOrder(middle)

        // Restore tree
        treap.root = merge(merge(before, middle), after)

        return result
    }
}
```

## Advantages

- **Simple implementation**: Easier than AVL or Red-Black trees
- **No rebalancing factors**: Just priorities and rotations
- **Efficient split/merge**: Natural O(log n) operations
- **Randomized balance**: High probability of good balance
- **Versatile**: Easy to extend (implicit keys, lazy propagation)

## Disadvantages

- **Probabilistic bounds**: No worst-case guarantees
- **Random number generation**: Need good random source
- **Priority storage**: Extra space per node
- **Not cache optimal**: Pointer-based structure
- **Non-deterministic**: Different runs may have different structure

## Comparison with Other BSTs

| Aspect           | Treap      | AVL Tree | Red-Black | Splay Tree |
|------------------|------------|----------|-----------|------------|
| Balance info     | Priority   | Height   | Color     | None       |
| Search (expected)| O(log n)   | O(log n) | O(log n)  | O(log n)*  |
| Insert           | O(log n)   | O(log n) | O(log n)  | O(log n)*  |
| Delete           | O(log n)   | O(log n) | O(log n)  | O(log n)*  |
| Split/Merge      | O(log n)   | O(n)     | O(n)      | O(log n)*  |
| Deterministic    | No         | Yes      | Yes       | Yes        |
| Implementation   | Simple     | Complex  | Complex   | Simple     |

*Amortized

## Common Pitfalls

- **Weak random priorities**: Use good random number generator
- **Forgetting size updates**: In implicit treaps
- **Split/merge order**: Must maintain BST invariant
- **Duplicate keys**: Need consistent handling
- **Stack overflow**: Deep recursion on degenerate trees

## Related Structures

- **Splay Tree**: Self-adjusting without randomization
- **Skip List**: Another randomized structure
- **AVL/Red-Black Trees**: Deterministic balanced trees
- **Cartesian Tree**: Special case where keys determine priorities
- **Zip Tree**: Recent simplification of treaps

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
