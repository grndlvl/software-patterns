# Splay Tree

## Overview

A Splay Tree is a self-adjusting binary search tree where recently accessed elements are moved to the root through a series of rotations called "splaying." This provides O(log n) amortized time for search, insert, and delete operations without storing any balance information in the nodes. The key insight is that frequently accessed elements stay near the root, providing excellent performance for workloads with temporal locality.

Splay trees were invented by Daniel Sleator and Robert Tarjan in 1985.

## Properties

- **Self-adjusting**: Restructures on every access
- **No balance information**: Simpler node structure than AVL/Red-Black
- **Amortized O(log n)**: Individual operations may be O(n), but amortized O(log n)
- **Working set property**: Recently accessed elements are fast to access again
- **Static optimality**: Performs within constant factor of optimal static tree

## Time Complexity

| Operation | Amortized | Worst Case | Notes                        |
|-----------|-----------|------------|------------------------------|
| Search    | O(log n)  | O(n)       | Splays accessed node to root |
| Insert    | O(log n)  | O(n)       | Insert then splay            |
| Delete    | O(log n)  | O(n)       | Splay then delete from root  |
| Splay     | O(log n)  | O(n)       | Core operation               |
| Split     | O(log n)  | O(n)       | Split at a key               |
| Join      | O(log n)  | O(n)       | Merge two trees              |

## Space Complexity

O(n) for n elements. No additional balance information per node.

## Structure

### Node Structure

```pseudocode
class SplayNode {
    key
    value         // Optional associated data
    left = null
    right = null
    parent = null // Optional, simplifies implementation
}

class SplayTree {
    root = null
}
```

## Splay Operation

The splay operation moves a node to the root through rotations. Three cases:

### Zig (Single Rotation)

When x is a child of the root:

```
    y               x
   / \             / \
  x   C    →      A   y
 / \                 / \
A   B               B   C
```

### Zig-Zig (Same Direction)

When x and its parent are both left children (or both right):

```
      z               x
     / \             / \
    y   D           A   y
   / \       →         / \
  x   C               B   z
 / \                     / \
A   B                   C   D
```

### Zig-Zag (Different Direction)

When x is a right child and its parent is a left child (or vice versa):

```
    z                 x
   / \               / \
  y   D             y   z
 / \       →       / \ / \
A   x             A  B C  D
   / \
  B   C
```

### Splay Implementation

```pseudocode
function splay(tree, x) {
    while x.parent != null {
        parent = x.parent
        grandparent = parent.parent

        if grandparent == null {
            // Zig: x is child of root
            if x == parent.left {
                rotateRight(tree, parent)
            } else {
                rotateLeft(tree, parent)
            }
        } else if x == parent.left and parent == grandparent.left {
            // Zig-zig: both left children
            rotateRight(tree, grandparent)
            rotateRight(tree, parent)
        } else if x == parent.right and parent == grandparent.right {
            // Zig-zig: both right children
            rotateLeft(tree, grandparent)
            rotateLeft(tree, parent)
        } else if x == parent.right and parent == grandparent.left {
            // Zig-zag: x is right, parent is left
            rotateLeft(tree, parent)
            rotateRight(tree, grandparent)
        } else {
            // Zig-zag: x is left, parent is right
            rotateRight(tree, parent)
            rotateLeft(tree, grandparent)
        }
    }

    tree.root = x
}

function rotateLeft(tree, x) {
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
}

function rotateRight(tree, x) {
    y = x.left
    x.left = y.right

    if y.right != null {
        y.right.parent = x
    }

    y.parent = x.parent

    if x.parent == null {
        tree.root = y
    } else if x == x.parent.right {
        x.parent.right = y
    } else {
        x.parent.left = y
    }

    y.right = x
    x.parent = y
}
```

## Operations

### Search

```pseudocode
function search(tree, key) {
    node = tree.root
    last = null

    while node != null {
        last = node
        if key == node.key {
            splay(tree, node)
            return node.value
        } else if key < node.key {
            node = node.left
        } else {
            node = node.right
        }
    }

    // Splay last accessed node even on failure
    if last != null {
        splay(tree, last)
    }

    return null  // Not found
}
```

### Insert

```pseudocode
function insert(tree, key, value) {
    if tree.root == null {
        tree.root = new SplayNode(key, value)
        return
    }

    // Search for position
    node = tree.root
    parent = null

    while node != null {
        parent = node
        if key == node.key {
            node.value = value  // Update existing
            splay(tree, node)
            return
        } else if key < node.key {
            node = node.left
        } else {
            node = node.right
        }
    }

    // Create new node
    newNode = new SplayNode(key, value)
    newNode.parent = parent

    if key < parent.key {
        parent.left = newNode
    } else {
        parent.right = newNode
    }

    splay(tree, newNode)
}
```

### Delete

```pseudocode
function delete(tree, key) {
    // Search and splay the node to root
    node = findNode(tree, key)

    if node == null {
        return false
    }

    splay(tree, node)

    // Now node is at root
    if node.left == null {
        tree.root = node.right
        if tree.root != null {
            tree.root.parent = null
        }
    } else if node.right == null {
        tree.root = node.left
        if tree.root != null {
            tree.root.parent = null
        }
    } else {
        // Join left and right subtrees
        leftTree = node.left
        rightTree = node.right

        leftTree.parent = null
        rightTree.parent = null

        // Splay max of left subtree
        maxLeft = leftTree
        while maxLeft.right != null {
            maxLeft = maxLeft.right
        }

        // Create temporary tree for left subtree
        tempTree = new SplayTree()
        tempTree.root = leftTree
        splay(tempTree, maxLeft)

        // Now maxLeft is root of left subtree and has no right child
        maxLeft.right = rightTree
        rightTree.parent = maxLeft

        tree.root = maxLeft
    }

    return true
}
```

### Split

```pseudocode
function split(tree, key) {
    // Split tree into two: left has keys < key, right has keys >= key
    if tree.root == null {
        return (null, null)
    }

    // Search for key (or closest)
    search(tree, key)

    // Root is now the searched node or closest
    if tree.root.key < key {
        // Split: root and left go to leftTree, right goes to rightTree
        leftTree = new SplayTree()
        rightTree = new SplayTree()

        leftTree.root = tree.root
        rightTree.root = tree.root.right

        if rightTree.root != null {
            rightTree.root.parent = null
        }
        tree.root.right = null

        return (leftTree, rightTree)
    } else {
        // Root key >= key
        leftTree = new SplayTree()
        rightTree = new SplayTree()

        rightTree.root = tree.root
        leftTree.root = tree.root.left

        if leftTree.root != null {
            leftTree.root.parent = null
        }
        tree.root.left = null

        return (leftTree, rightTree)
    }
}
```

### Join

```pseudocode
function join(leftTree, rightTree) {
    // Join two trees where all keys in leftTree < all keys in rightTree
    if leftTree.root == null {
        return rightTree
    }
    if rightTree.root == null {
        return leftTree
    }

    // Splay max of left tree
    maxLeft = leftTree.root
    while maxLeft.right != null {
        maxLeft = maxLeft.right
    }
    splay(leftTree, maxLeft)

    // maxLeft is now root with no right child
    maxLeft.right = rightTree.root
    rightTree.root.parent = maxLeft

    return leftTree
}
```

## Use Cases

### LRU Cache Enhancement

```pseudocode
class SplayCache {
    tree = new SplayTree()
    maxSize

    function get(key) {
        // Access moves item to root (most recently used)
        return tree.search(key)
    }

    function put(key, value) {
        tree.insert(key, value)

        // Evict least recently used (deepest nodes)
        while tree.size() > maxSize {
            evictDeepest(tree)
        }
    }
}
```

### Adaptive Data Structure

```pseudocode
class AdaptiveIndex {
    tree = new SplayTree()

    function query(key) {
        // Frequently queried keys naturally stay near root
        return tree.search(key)
    }

    // No rebalancing needed - structure adapts automatically
}
```

## Advantages

- **Simple implementation**: No balance factors or colors to maintain
- **Self-optimizing**: Frequently accessed items stay near root
- **Good cache behavior**: Working set stays near top
- **Amortized efficiency**: O(log n) amortized despite simple structure
- **Sequential access theorem**: Accessing n items in sorted order takes O(n)
- **Flexible operations**: Easy to implement split and join

## Disadvantages

- **No worst-case guarantees**: Individual operations can be O(n)
- **Always modifies structure**: Even reads modify the tree
- **Not thread-safe friendly**: Structure changes on every access
- **Poor for uniform access**: If all keys accessed equally often
- **Splaying overhead**: Extra work even for simple accesses

## Comparison with Other BSTs

| Aspect           | Splay Tree | AVL Tree | Red-Black Tree |
|------------------|------------|----------|----------------|
| Balance info     | None       | Height   | Color          |
| Search (worst)   | O(n)       | O(log n) | O(log n)       |
| Search (amort)   | O(log n)   | O(log n) | O(log n)       |
| Insert (worst)   | O(n)       | O(log n) | O(log n)       |
| Insert (amort)   | O(log n)   | O(log n) | O(log n)       |
| Read-only search | Modifies   | Pure     | Pure           |
| Sequential access| O(n) total | O(n log n)| O(n log n)    |
| Implementation   | Simpler    | Complex  | Complex        |

## Common Pitfalls

- **Forgetting to splay on search miss**: Should splay last accessed node
- **Incorrect rotation order**: Zig-zig vs zig-zag matters
- **Parent pointer bugs**: Must update parent pointers correctly
- **Not handling single-child cases**: In delete operation
- **Ignoring amortized nature**: Don't expect O(log n) for every single op

## Related Structures

- **Treap**: Randomized BST with heap property
- **Red-Black Tree**: Balanced with O(log n) worst-case
- **AVL Tree**: Strictly balanced with O(log n) worst-case
- **Link-Cut Tree**: Splay trees for dynamic tree problems
- **Tango Tree**: O(log log n) competitive ratio

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
