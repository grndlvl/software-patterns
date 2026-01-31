# Order-Statistic Tree

## Overview

An Order-Statistic Tree is an augmented self-balancing binary search tree that supports efficient rank-based operations. In addition to standard BST operations, it can find the k-th smallest element and determine the rank of any element in O(log n) time. This is achieved by storing the size of each subtree at every node.

Order-statistic trees are used when you need both sorted-order access and position-based queries, such as finding medians, percentiles, or maintaining a dynamic sorted list with indexed access.

## Properties

- **Augmented BST**: Each node stores the size of its subtree
- **Self-balancing**: Built on Red-Black or AVL tree
- **Size invariant**: node.size = node.left.size + node.right.size + 1
- **Rank support**: Can find element by rank or rank by element
- **Dynamic**: Supports insertions and deletions while maintaining order

## Time Complexity

| Operation      | Time      | Notes                           |
|----------------|-----------|--------------------------------|
| Insert         | O(log n)  | Standard BST + update sizes    |
| Delete         | O(log n)  | Standard BST + update sizes    |
| Search         | O(log n)  | Standard BST search            |
| Select(k)      | O(log n)  | Find k-th smallest element     |
| Rank(x)        | O(log n)  | Find rank of element x         |
| Min/Max        | O(log n)  | Standard BST operations        |
| Successor      | O(log n)  | Standard BST operations        |
| Predecessor    | O(log n)  | Standard BST operations        |

## Space Complexity

O(n) for n elements. Each node stores key, size, and standard BST pointers plus balancing info.

## Structure

### Node Structure

```pseudocode
class OrderStatisticNode {
    key              // Element value
    size = 1         // Size of subtree rooted at this node
    left = null      // Left child
    right = null     // Right child
    parent = null    // Parent pointer
    // Additional fields for balancing (color for Red-Black, height for AVL)
}

// Helper function to get size safely
function getSize(node) {
    if node == null {
        return 0
    }
    return node.size
}
```

## Operations

### Select (Find k-th Smallest)

```pseudocode
function select(tree, k) {
    // Find the k-th smallest element (1-indexed)
    if k < 1 or k > getSize(tree.root) {
        return null  // Invalid rank
    }

    return selectHelper(tree.root, k)
}

function selectHelper(node, k) {
    // r = rank of node within its subtree
    r = getSize(node.left) + 1

    if k == r {
        return node  // Found it
    } else if k < r {
        // k-th smallest is in left subtree
        return selectHelper(node.left, k)
    } else {
        // k-th smallest is in right subtree
        // Subtract elements in left subtree and current node
        return selectHelper(node.right, k - r)
    }
}
```

### Rank (Find Position of Element)

```pseudocode
function rank(tree, key) {
    // Find rank of element with given key (1-indexed)
    return rankHelper(tree.root, key)
}

function rankHelper(node, key) {
    if node == null {
        return 0  // Element not found
    }

    if key == node.key {
        // Rank is left subtree size + 1
        return getSize(node.left) + 1
    } else if key < node.key {
        // Element is in left subtree, rank is same there
        return rankHelper(node.left, key)
    } else {
        // Element is in right subtree
        // Add left subtree size + 1 (current node) + rank in right
        return getSize(node.left) + 1 + rankHelper(node.right, key)
    }
}

// Alternative: Rank using parent pointers (bottom-up)
function rankBottomUp(tree, node) {
    r = getSize(node.left) + 1
    current = node

    while current.parent != null {
        if current == current.parent.right {
            // Current is right child, add left sibling size + 1
            r = r + getSize(current.parent.left) + 1
        }
        current = current.parent
    }

    return r
}
```

### Insert

```pseudocode
function insert(tree, key) {
    node = new OrderStatisticNode()
    node.key = key
    node.size = 1

    if tree.root == null {
        tree.root = node
        return node
    }

    // Standard BST insert
    current = tree.root
    parent = null

    while current != null {
        parent = current
        current.size = current.size + 1  // Increment size on path

        if key < current.key {
            current = current.left
        } else {
            current = current.right
        }
    }

    node.parent = parent
    if key < parent.key {
        parent.left = node
    } else {
        parent.right = node
    }

    // Rebalance (Red-Black or AVL)
    rebalanceAfterInsert(tree, node)

    return node
}
```

### Delete

```pseudocode
function delete(tree, key) {
    node = search(tree.root, key)

    if node == null {
        return false
    }

    // Decrement sizes on path from root to node
    decrementSizesOnPath(tree.root, key)

    // Standard BST delete with balancing
    if node.left == null {
        transplant(tree, node, node.right)
    } else if node.right == null {
        transplant(tree, node, node.left)
    } else {
        // Find successor
        successor = minimum(node.right)

        if successor.parent != node {
            // Decrement sizes from node.right to successor
            decrementSizesBetween(node.right, successor)
            transplant(tree, successor, successor.right)
            successor.right = node.right
            successor.right.parent = successor
        }

        transplant(tree, node, successor)
        successor.left = node.left
        successor.left.parent = successor
        successor.size = node.size - 1  // Successor takes node's size - 1
    }

    // Rebalance and fix sizes
    rebalanceAfterDelete(tree, node)

    return true
}

function decrementSizesOnPath(node, key) {
    while node != null {
        node.size = node.size - 1
        if key < node.key {
            node = node.left
        } else if key > node.key {
            node = node.right
        } else {
            break
        }
    }
}
```

### Update Size After Rotation

```pseudocode
// Left rotation: x becomes child of y
function leftRotate(tree, x) {
    y = x.right
    x.right = y.left

    if y.left != null {
        y.left.parent = x
    }

    y.parent = x.parent
    if x.parent == null {
        tree.root = y
    } else if x == x.parent.left {
        x.parent.left = y
    } else {
        x.parent.right = y
    }

    y.left = x
    x.parent = y

    // Fix sizes: x's size changes, then y's size changes
    y.size = x.size
    x.size = getSize(x.left) + getSize(x.right) + 1
}

// Right rotation: y becomes child of x
function rightRotate(tree, y) {
    x = y.left
    y.left = x.right

    if x.right != null {
        x.right.parent = y
    }

    x.parent = y.parent
    if y.parent == null {
        tree.root = x
    } else if y == y.parent.left {
        y.parent.left = x
    } else {
        y.parent.right = x
    }

    x.right = y
    y.parent = x

    // Fix sizes
    x.size = y.size
    y.size = getSize(y.left) + getSize(y.right) + 1
}
```

## Advanced Operations

### Range Count

```pseudocode
function countInRange(tree, low, high) {
    // Count elements in [low, high]
    if low > high {
        return 0
    }

    rankHigh = rank(tree, high)
    if search(tree.root, high) == null {
        // high not in tree, find predecessor's rank
        rankHigh = countLessOrEqual(tree.root, high)
    }

    rankLow = rank(tree, low)
    if search(tree.root, low) == null {
        rankLow = countLessOrEqual(tree.root, low - 1) + 1
    }

    return rankHigh - rankLow + 1
}

function countLessOrEqual(node, key) {
    if node == null {
        return 0
    }

    if key < node.key {
        return countLessOrEqual(node.left, key)
    } else if key > node.key {
        return getSize(node.left) + 1 + countLessOrEqual(node.right, key)
    } else {
        return getSize(node.left) + 1
    }
}
```

### Find Median

```pseudocode
function findMedian(tree) {
    n = getSize(tree.root)

    if n == 0 {
        return null
    }

    if n % 2 == 1 {
        // Odd count: return middle element
        return select(tree, (n + 1) / 2).key
    } else {
        // Even count: return average of two middle elements
        lower = select(tree, n / 2).key
        upper = select(tree, n / 2 + 1).key
        return (lower + upper) / 2
    }
}
```

### Find Percentile

```pseudocode
function findPercentile(tree, p) {
    // Find element at percentile p (0-100)
    n = getSize(tree.root)
    k = max(1, ceil(p * n / 100))
    return select(tree, k).key
}
```

## Use Cases

### Running Median

```pseudocode
class RunningMedian {
    tree = new OrderStatisticTree()

    function add(value) {
        tree.insert(value)
    }

    function getMedian() {
        return findMedian(tree)
    }

    function remove(value) {
        tree.delete(value)
    }
}
```

### Leaderboard

```pseudocode
class Leaderboard {
    scores = new OrderStatisticTree()  // Stores (score, playerId) pairs

    function addScore(playerId, score) {
        scores.insert((score, playerId))
    }

    function getTopK(k) {
        result = []
        n = scores.size()
        for i from n down to max(1, n - k + 1) {
            result.append(scores.select(i))
        }
        return result
    }

    function getPlayerRank(playerId, score) {
        // Higher score = better rank
        totalRank = scores.rank((score, playerId))
        return scores.size() - totalRank + 1
    }

    function getPlayersInRankRange(startRank, endRank) {
        result = []
        n = scores.size()
        for i from n - startRank + 1 down to n - endRank + 1 {
            if i >= 1 and i <= n {
                result.append(scores.select(i))
            }
        }
        return result
    }
}
```

### Database Indexing with Position

```pseudocode
class IndexedSet {
    tree = new OrderStatisticTree()

    function add(element) {
        tree.insert(element)
    }

    function getByIndex(index) {
        // 0-indexed access
        return tree.select(index + 1)
    }

    function indexOf(element) {
        // Returns 0-indexed position
        return tree.rank(element) - 1
    }

    function slice(start, end) {
        // Return elements in index range [start, end)
        result = []
        for i from start to end - 1 {
            result.append(getByIndex(i))
        }
        return result
    }
}
```

## Advantages

- **O(log n) rank operations**: Fast select and rank queries
- **Dynamic**: Supports insertions and deletions
- **Sorted order**: Maintains sorted elements
- **All BST operations**: Standard search, min, max, successor, predecessor
- **Space efficient**: Only adds one integer per node

## Disadvantages

- **Implementation complexity**: Must maintain sizes during all operations
- **Rotation overhead**: Size updates during rebalancing
- **No efficient range updates**: Modifying ranges is O(k log n)
- **Single key lookups**: Cannot have duplicate keys (or need modification)
- **Memory overhead**: Extra size field per node

## Comparison with Alternatives

| Aspect           | Order-Statistic Tree | Sorted Array | Skip List    |
|------------------|---------------------|--------------|--------------|
| Select(k)        | O(log n)            | O(1)         | O(log n)     |
| Rank(x)          | O(log n)            | O(log n)*    | O(log n)     |
| Insert           | O(log n)            | O(n)         | O(log n)     |
| Delete           | O(log n)            | O(n)         | O(log n)     |
| Search           | O(log n)            | O(log n)     | O(log n)     |
| Space            | O(n)                | O(n)         | O(n log n)   |
| Implementation   | Complex             | Simple       | Moderate     |

*Using binary search

## Common Pitfalls

- **Forgetting to update sizes**: After insert, delete, or rotation
- **Off-by-one in rank**: Be consistent with 0-indexed vs 1-indexed
- **Not handling null children**: Size of null is 0, not undefined
- **Incorrect size during delete**: Must update all affected nodes
- **Rotation size updates**: Order matters when updating sizes
- **Duplicate handling**: Standard BST doesn't handle duplicates well

## Related Structures

- **Interval Tree**: Augmented BST for interval overlap queries
- **Fenwick Tree**: For prefix sums with O(log n) updates
- **Segment Tree**: For range queries and updates
- **Skip List**: Alternative with similar complexity, simpler rotations
- **Treap**: Randomized BST that can be augmented similarly

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
