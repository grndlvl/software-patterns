# Red-Black Tree

## Overview

A Red-Black tree is a self-balancing binary search tree where each node has an extra bit for color (red or black). The coloring rules ensure the tree remains approximately balanced, guaranteeing O(log n) operations. Red-Black trees trade stricter balance (like AVL) for fewer rotations during modifications.

## Properties

Every Red-Black tree must satisfy these five properties:

1. **Node Color**: Every node is either red or black
2. **Root Property**: The root is always black
3. **Leaf Property**: All null leaves (NIL) are black
4. **Red Property**: Red nodes cannot have red children (no two reds in a row)
5. **Black Height**: Every path from a node to its descendant NIL nodes has the same number of black nodes

### Key Insight

The constraints ensure that the longest path is at most twice the shortest path, guaranteeing O(log n) height.

```
Black Height: Number of black nodes from node to leaf (not counting the node itself)

Maximum height: 2 * log2(n + 1)
```

## Time Complexity

| Operation | Average   | Worst     |
|-----------|-----------|-----------|
| Search    | O(log n)  | O(log n)  |
| Insert    | O(log n)  | O(log n)  |
| Delete    | O(log n)  | O(log n)  |
| Min/Max   | O(log n)  | O(log n)  |

## Space Complexity

O(n) for n nodes. One extra bit per node for color (often stored in a byte or pointer bit).

## Operations

### Node Structure

```pseudocode
class RBNode {
    data
    color       // RED or BLACK
    left = NIL
    right = NIL
    parent = NIL

    function constructor(data) {
        this.data = data
        this.color = RED  // New nodes start as red
    }
}

// Sentinel NIL node (simplifies edge cases)
NIL = new RBNode(null)
NIL.color = BLACK
```

### Rotations

```pseudocode
function rotateLeft(tree, x) {
    y = x.right
    x.right = y.left

    if y.left != NIL {
        y.left.parent = x
    }

    y.parent = x.parent

    if x.parent == NIL {
        tree.root = y
    } else if x == x.parent.left {
        x.parent.left = y
    } else {
        x.parent.right = y
    }

    y.left = x
    x.parent = y
}

function rotateRight(tree, y) {
    x = y.left
    y.left = x.right

    if x.right != NIL {
        x.right.parent = y
    }

    x.parent = y.parent

    if y.parent == NIL {
        tree.root = x
    } else if y == y.parent.right {
        y.parent.right = x
    } else {
        y.parent.left = x
    }

    x.right = y
    y.parent = x
}
```

### Insert

```pseudocode
function insert(tree, key) {
    newNode = new RBNode(key)
    newNode.left = NIL
    newNode.right = NIL

    // Standard BST insert
    parent = NIL
    current = tree.root

    while current != NIL {
        parent = current
        if key < current.data {
            current = current.left
        } else {
            current = current.right
        }
    }

    newNode.parent = parent

    if parent == NIL {
        tree.root = newNode
    } else if key < parent.data {
        parent.left = newNode
    } else {
        parent.right = newNode
    }

    newNode.color = RED

    // Fix Red-Black violations
    insertFixup(tree, newNode)
}

function insertFixup(tree, node) {
    while node.parent.color == RED {
        if node.parent == node.parent.parent.left {
            uncle = node.parent.parent.right

            // Case 1: Uncle is red - recolor
            if uncle.color == RED {
                node.parent.color = BLACK
                uncle.color = BLACK
                node.parent.parent.color = RED
                node = node.parent.parent
            } else {
                // Case 2: Uncle is black, node is right child
                if node == node.parent.right {
                    node = node.parent
                    rotateLeft(tree, node)
                }
                // Case 3: Uncle is black, node is left child
                node.parent.color = BLACK
                node.parent.parent.color = RED
                rotateRight(tree, node.parent.parent)
            }
        } else {
            // Mirror cases (parent is right child)
            uncle = node.parent.parent.left

            if uncle.color == RED {
                node.parent.color = BLACK
                uncle.color = BLACK
                node.parent.parent.color = RED
                node = node.parent.parent
            } else {
                if node == node.parent.left {
                    node = node.parent
                    rotateRight(tree, node)
                }
                node.parent.color = BLACK
                node.parent.parent.color = RED
                rotateLeft(tree, node.parent.parent)
            }
        }
    }

    tree.root.color = BLACK
}
```

### Delete

```pseudocode
function delete(tree, key) {
    node = search(tree.root, key)
    if node == NIL {
        return
    }

    y = node
    yOriginalColor = y.color

    if node.left == NIL {
        x = node.right
        transplant(tree, node, node.right)
    } else if node.right == NIL {
        x = node.left
        transplant(tree, node, node.left)
    } else {
        y = minimum(node.right)
        yOriginalColor = y.color
        x = y.right

        if y.parent == node {
            x.parent = y
        } else {
            transplant(tree, y, y.right)
            y.right = node.right
            y.right.parent = y
        }

        transplant(tree, node, y)
        y.left = node.left
        y.left.parent = y
        y.color = node.color
    }

    if yOriginalColor == BLACK {
        deleteFixup(tree, x)
    }
}

function transplant(tree, u, v) {
    if u.parent == NIL {
        tree.root = v
    } else if u == u.parent.left {
        u.parent.left = v
    } else {
        u.parent.right = v
    }
    v.parent = u.parent
}

function deleteFixup(tree, x) {
    while x != tree.root and x.color == BLACK {
        if x == x.parent.left {
            sibling = x.parent.right

            // Case 1: Sibling is red
            if sibling.color == RED {
                sibling.color = BLACK
                x.parent.color = RED
                rotateLeft(tree, x.parent)
                sibling = x.parent.right
            }

            // Case 2: Sibling's children are both black
            if sibling.left.color == BLACK and sibling.right.color == BLACK {
                sibling.color = RED
                x = x.parent
            } else {
                // Case 3: Sibling's right child is black
                if sibling.right.color == BLACK {
                    sibling.left.color = BLACK
                    sibling.color = RED
                    rotateRight(tree, sibling)
                    sibling = x.parent.right
                }
                // Case 4: Sibling's right child is red
                sibling.color = x.parent.color
                x.parent.color = BLACK
                sibling.right.color = BLACK
                rotateLeft(tree, x.parent)
                x = tree.root
            }
        } else {
            // Mirror cases
            sibling = x.parent.left

            if sibling.color == RED {
                sibling.color = BLACK
                x.parent.color = RED
                rotateRight(tree, x.parent)
                sibling = x.parent.left
            }

            if sibling.right.color == BLACK and sibling.left.color == BLACK {
                sibling.color = RED
                x = x.parent
            } else {
                if sibling.left.color == BLACK {
                    sibling.right.color = BLACK
                    sibling.color = RED
                    rotateLeft(tree, sibling)
                    sibling = x.parent.left
                }
                sibling.color = x.parent.color
                x.parent.color = BLACK
                sibling.left.color = BLACK
                rotateRight(tree, x.parent)
                x = tree.root
            }
        }
    }

    x.color = BLACK
}
```

### Search

```pseudocode
function search(node, key) {
    while node != NIL and key != node.data {
        if key < node.data {
            node = node.left
        } else {
            node = node.right
        }
    }
    return node
}
```

## Implementation

```pseudocode
class RedBlackTree {
    root = NIL

    function insert(key) {
        // Implementation from above
    }

    function delete(key) {
        // Implementation from above
    }

    function search(key) {
        return searchNode(this.root, key)
    }

    function searchNode(node, key) {
        while node != NIL and key != node.data {
            if key < node.data {
                node = node.left
            } else {
                node = node.right
            }
        }
        return node
    }

    function minimum(node) {
        while node.left != NIL {
            node = node.left
        }
        return node
    }

    function maximum(node) {
        while node.right != NIL {
            node = node.right
        }
        return node
    }

    function contains(key) {
        return search(key) != NIL
    }
}
```

## Insert Cases Summary

| Case | Condition | Action |
|------|-----------|--------|
| 1 | Uncle is red | Recolor parent, uncle, grandparent |
| 2 | Uncle black, node is inner child | Rotate to make outer child (→ Case 3) |
| 3 | Uncle black, node is outer child | Rotate grandparent, recolor |

## Delete Cases Summary

| Case | Condition | Action |
|------|-----------|--------|
| 1 | Sibling is red | Rotate parent, recolor |
| 2 | Sibling black, both nephews black | Recolor sibling, move up |
| 3 | Sibling black, far nephew black | Rotate sibling (→ Case 4) |
| 4 | Sibling black, far nephew red | Rotate parent, recolor |

## Use Cases

- **Standard library implementations**: Java TreeMap, C++ std::map
- **Linux kernel**: Completely Fair Scheduler, memory management
- **Database systems**: Index structures
- **File systems**: Directory indexing
- **Symbol tables**: Compiler implementations
- **Interval trees**: When modifications are frequent

## Advantages

- **Guaranteed O(log n)**: All operations have logarithmic worst case
- **Fewer rotations**: At most 2 rotations per insert, 3 per delete
- **Good for modifications**: Better than AVL for insert-heavy workloads
- **Widely implemented**: Battle-tested in many standard libraries
- **Simpler than AVL delete**: Conceptually, though code is complex

## Disadvantages

- **Complex implementation**: Many cases to handle
- **Less balanced than AVL**: Height can be up to 2x optimal
- **Slower lookups**: Than AVL due to less strict balance
- **Extra storage**: Color bit per node
- **Hard to understand**: Five properties and multiple cases

## Comparison with Alternatives

| Aspect          | Red-Black   | AVL Tree    | Splay Tree  | B-Tree      |
|-----------------|-------------|-------------|-------------|-------------|
| Search          | O(log n)    | O(log n)    | O(log n)*   | O(log n)    |
| Insert          | O(log n)    | O(log n)    | O(log n)*   | O(log n)    |
| Delete          | O(log n)    | O(log n)    | O(log n)*   | O(log n)    |
| Max height      | 2 log n     | 1.44 log n  | O(n)        | logB n      |
| Rotations/insert| ≤ 2         | O(log n)    | O(log n)    | 0           |
| Best for        | General use | Lookups     | Locality    | Disk I/O    |

*Amortized

## Common Pitfalls

- **Forgetting NIL sentinel**: Simplifies code but easy to forget
- **Color assignment**: New nodes must be red
- **Root color**: Must always be black after operations
- **Case confusion**: Many similar cases, easy to mix up
- **Parent pointers**: Must maintain during rotations
- **Mirror cases**: Forgetting symmetric cases for right/left

## Related Structures

- **AVL Tree**: Stricter balancing, faster lookups
- **2-3-4 Tree**: Isomorphic to Red-Black tree
- **Left-Leaning Red-Black**: Simplified variant
- **AA Tree**: Simpler Red-Black variant
- **B-Tree**: Multi-way balanced tree

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
