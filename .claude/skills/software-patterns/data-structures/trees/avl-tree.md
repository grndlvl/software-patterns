# AVL Tree

## Overview

An AVL tree (named after inventors Adelson-Velsky and Landis) is a self-balancing binary search tree where the heights of the left and right subtrees of any node differ by at most one. This balance guarantee ensures O(log n) operations in all cases, eliminating the worst-case O(n) of unbalanced BSTs.

## Properties

- **BST property**: Left < Parent < Right
- **Balance factor**: height(left) - height(right) in {-1, 0, 1}
- **Height-balanced**: Height is always O(log n)
- **Self-balancing**: Automatically rebalances on insert/delete
- **Strictly balanced**: More balanced than Red-Black trees

### Balance Factor

```
Balance Factor = Height(Left Subtree) - Height(Right Subtree)

Valid values: -1, 0, +1
If |BF| > 1: Tree needs rebalancing
```

## Time Complexity

| Operation | Average   | Worst     |
|-----------|-----------|-----------|
| Search    | O(log n)  | O(log n)  |
| Insert    | O(log n)  | O(log n)  |
| Delete    | O(log n)  | O(log n)  |
| Min/Max   | O(log n)  | O(log n)  |

## Space Complexity

O(n) for n nodes. Each node stores height (or balance factor) requiring O(1) extra space per node.

## Operations

### Node Structure

```pseudocode
class AVLNode {
    data
    left = null
    right = null
    height = 1  // Height of subtree rooted at this node

    function constructor(data) {
        this.data = data
    }
}
```

### Helper Functions

```pseudocode
function getHeight(node) {
    if node == null {
        return 0
    }
    return node.height
}

function getBalanceFactor(node) {
    if node == null {
        return 0
    }
    return getHeight(node.left) - getHeight(node.right)
}

function updateHeight(node) {
    node.height = 1 + max(getHeight(node.left), getHeight(node.right))
}
```

### Rotations

#### Right Rotation (LL Case)

```pseudocode
//       y                              x
//      / \     Right Rotation         / \
//     x   C    ------------->        A   y
//    / \                                / \
//   A   B                              B   C

function rotateRight(y) {
    x = y.left
    B = x.right

    // Perform rotation
    x.right = y
    y.left = B

    // Update heights
    updateHeight(y)
    updateHeight(x)

    return x  // New root
}
```

#### Left Rotation (RR Case)

```pseudocode
//     x                               y
//    / \      Left Rotation          / \
//   A   y     ------------->        x   C
//      / \                         / \
//     B   C                       A   B

function rotateLeft(x) {
    y = x.right
    B = y.left

    // Perform rotation
    y.left = x
    x.right = B

    // Update heights
    updateHeight(x)
    updateHeight(y)

    return y  // New root
}
```

### Rebalancing

```pseudocode
function rebalance(node) {
    updateHeight(node)
    balance = getBalanceFactor(node)

    // Left Heavy (balance > 1)
    if balance > 1 {
        // Left-Left Case
        if getBalanceFactor(node.left) >= 0 {
            return rotateRight(node)
        }
        // Left-Right Case
        else {
            node.left = rotateLeft(node.left)
            return rotateRight(node)
        }
    }

    // Right Heavy (balance < -1)
    if balance < -1 {
        // Right-Right Case
        if getBalanceFactor(node.right) <= 0 {
            return rotateLeft(node)
        }
        // Right-Left Case
        else {
            node.right = rotateRight(node.right)
            return rotateLeft(node)
        }
    }

    return node  // Already balanced
}
```

### Insert

```pseudocode
function insert(node, key) {
    // Standard BST insert
    if node == null {
        return new AVLNode(key)
    }

    if key < node.data {
        node.left = insert(node.left, key)
    } else if key > node.data {
        node.right = insert(node.right, key)
    } else {
        return node  // Duplicate keys not allowed
    }

    // Rebalance and return
    return rebalance(node)
}
```

### Delete

```pseudocode
function delete(node, key) {
    // Standard BST delete
    if node == null {
        return null
    }

    if key < node.data {
        node.left = delete(node.left, key)
    } else if key > node.data {
        node.right = delete(node.right, key)
    } else {
        // Found node to delete

        // Case 1 & 2: Node with one or no child
        if node.left == null {
            return node.right
        } else if node.right == null {
            return node.left
        }

        // Case 3: Node with two children
        // Get inorder successor (smallest in right subtree)
        successor = findMin(node.right)
        node.data = successor.data
        node.right = delete(node.right, successor.data)
    }

    // Rebalance and return
    return rebalance(node)
}

function findMin(node) {
    while node.left != null {
        node = node.left
    }
    return node
}
```

### Search

```pseudocode
function search(node, key) {
    if node == null {
        return null
    }

    if key == node.data {
        return node
    } else if key < node.data {
        return search(node.left, key)
    } else {
        return search(node.right, key)
    }
}
```

## Implementation

```pseudocode
class AVLTree {
    root = null

    function insert(key) {
        this.root = insertNode(this.root, key)
    }

    function insertNode(node, key) {
        if node == null {
            return new AVLNode(key)
        }

        if key < node.data {
            node.left = insertNode(node.left, key)
        } else if key > node.data {
            node.right = insertNode(node.right, key)
        } else {
            return node
        }

        return rebalance(node)
    }

    function delete(key) {
        this.root = deleteNode(this.root, key)
    }

    function deleteNode(node, key) {
        if node == null {
            return null
        }

        if key < node.data {
            node.left = deleteNode(node.left, key)
        } else if key > node.data {
            node.right = deleteNode(node.right, key)
        } else {
            if node.left == null {
                return node.right
            } else if node.right == null {
                return node.left
            }

            successor = findMinNode(node.right)
            node.data = successor.data
            node.right = deleteNode(node.right, successor.data)
        }

        return rebalance(node)
    }

    function search(key) {
        return searchNode(this.root, key)
    }

    function searchNode(node, key) {
        if node == null or node.data == key {
            return node
        }

        if key < node.data {
            return searchNode(node.left, key)
        }
        return searchNode(node.right, key)
    }

    function rebalance(node) {
        updateHeight(node)
        balance = getBalanceFactor(node)

        if balance > 1 {
            if getBalanceFactor(node.left) < 0 {
                node.left = rotateLeft(node.left)
            }
            return rotateRight(node)
        }

        if balance < -1 {
            if getBalanceFactor(node.right) > 0 {
                node.right = rotateRight(node.right)
            }
            return rotateLeft(node)
        }

        return node
    }

    function rotateRight(y) {
        x = y.left
        B = x.right

        x.right = y
        y.left = B

        updateHeight(y)
        updateHeight(x)

        return x
    }

    function rotateLeft(x) {
        y = x.right
        B = y.left

        y.left = x
        x.right = B

        updateHeight(x)
        updateHeight(y)

        return y
    }

    function getHeight(node) {
        return node == null ? 0 : node.height
    }

    function getBalanceFactor(node) {
        return node == null ? 0 : getHeight(node.left) - getHeight(node.right)
    }

    function updateHeight(node) {
        node.height = 1 + max(getHeight(node.left), getHeight(node.right))
    }

    function findMinNode(node) {
        while node.left != null {
            node = node.left
        }
        return node
    }
}
```

## Rotation Cases Summary

| Case        | Condition                      | Rotation              |
|-------------|--------------------------------|-----------------------|
| Left-Left   | BF > 1 and BF(left) >= 0       | Right rotation        |
| Left-Right  | BF > 1 and BF(left) < 0        | Left-Right rotation   |
| Right-Right | BF < -1 and BF(right) <= 0     | Left rotation         |
| Right-Left  | BF < -1 and BF(right) > 0      | Right-Left rotation   |

## Use Cases

- **Database indexing**: When guaranteed O(log n) is critical
- **In-memory databases**: Where reads are frequent
- **Spell checkers**: Fast dictionary lookups
- **Symbol tables**: Compiler implementations
- **Ordered statistics**: Finding kth element, rank queries
- **Time-critical applications**: Where worst-case matters

## Advantages

- **Guaranteed O(log n)**: All operations have logarithmic worst case
- **Strictly balanced**: More balanced than Red-Black trees
- **Faster lookups**: Better for read-heavy workloads
- **Predictable performance**: No degradation with any input
- **Ordered operations**: Supports range queries, min/max

## Disadvantages

- **More rotations**: Stricter balance means more rebalancing
- **Slower inserts/deletes**: Compared to Red-Black trees
- **More complex**: Than simple BST
- **Extra storage**: Height field per node
- **Not cache-optimal**: Pointer-based structure

## Comparison with Alternatives

| Aspect          | AVL Tree    | Red-Black   | BST         | B-Tree      |
|-----------------|-------------|-------------|-------------|-------------|
| Search          | O(log n)    | O(log n)    | O(n) worst  | O(log n)    |
| Insert          | O(log n)    | O(log n)    | O(n) worst  | O(log n)    |
| Balance         | Strict      | Relaxed     | None        | Strict      |
| Height          | 1.44 log n  | 2 log n     | n worst     | logB n      |
| Rotations/op    | O(log n)    | O(1) amort. | 0           | O(log n)    |
| Best for        | Lookups     | Inserts     | Simple use  | Disk I/O    |

## Common Pitfalls

- **Height update order**: Update children before parent
- **Rotation direction**: Confusing left vs right rotation cases
- **Balance factor sign**: Consistent definition (left - right or right - left)
- **Forgetting to return**: New root after rotation
- **Double rotation**: Not applying both rotations in LR/RL cases
- **Deletion complexity**: Multiple cases with rebalancing

## Related Structures

- **Red-Black Tree**: Less strict balancing, fewer rotations
- **Splay Tree**: Self-adjusting, no explicit balance
- **2-3 Tree**: Conceptual basis for Red-Black trees
- **B-Tree**: Multi-way balanced tree for disk storage
- **Weight-balanced Tree**: Balance by subtree sizes

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
