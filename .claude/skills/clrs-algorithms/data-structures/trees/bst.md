# Binary Search Tree (BST)

## Overview

A Binary Search Tree is a binary tree with the ordering property: for every node, all values in its left subtree are less than the node's value, and all values in its right subtree are greater. This property enables efficient searching, insertion, and deletion operations.

## Properties

- **Binary tree structure**: Each node has at most two children
- **Ordering invariant**: left < parent < right (for all subtrees)
- **Unique values**: Typically no duplicates (or consistent placement rule)
- **Inorder traversal**: Yields sorted sequence
- **Recursive structure**: Each subtree is also a valid BST

## Time Complexity

| Operation | Average     | Worst (Unbalanced) |
|-----------|-------------|--------------------|
| Search    | O(log n)    | O(n)               |
| Insert    | O(log n)    | O(n)               |
| Delete    | O(log n)    | O(n)               |
| Min/Max   | O(log n)    | O(n)               |
| Successor | O(log n)    | O(n)               |

*Worst case occurs with degenerate tree (essentially a linked list)*

## Space Complexity

O(n) for n nodes. Recursive operations use O(h) stack space where h is height:
- Balanced: h = O(log n)
- Unbalanced: h = O(n)

## Operations

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

// Iterative version
function searchIterative(root, key) {
    current = root

    while current != null {
        if key == current.data {
            return current
        } else if key < current.data {
            current = current.left
        } else {
            current = current.right
        }
    }

    return null
}
```

### Insert

```pseudocode
function insert(node, key) {
    if node == null {
        return new TreeNode(key)
    }

    if key < node.data {
        node.left = insert(node.left, key)
    } else if key > node.data {
        node.right = insert(node.right, key)
    }
    // If key == node.data, duplicate - ignore or handle as needed

    return node
}

// Iterative version
function insertIterative(root, key) {
    newNode = new TreeNode(key)

    if root == null {
        return newNode
    }

    current = root
    parent = null

    while current != null {
        parent = current
        if key < current.data {
            current = current.left
        } else if key > current.data {
            current = current.right
        } else {
            return root  // Duplicate
        }
    }

    if key < parent.data {
        parent.left = newNode
    } else {
        parent.right = newNode
    }

    return root
}
```

### Delete

```pseudocode
function delete(node, key) {
    if node == null {
        return null
    }

    if key < node.data {
        node.left = delete(node.left, key)
    } else if key > node.data {
        node.right = delete(node.right, key)
    } else {
        // Found the node to delete

        // Case 1: No children (leaf)
        if node.left == null and node.right == null {
            return null
        }

        // Case 2: One child
        if node.left == null {
            return node.right
        }
        if node.right == null {
            return node.left
        }

        // Case 3: Two children
        // Replace with inorder successor (smallest in right subtree)
        successor = findMin(node.right)
        node.data = successor.data
        node.right = delete(node.right, successor.data)
    }

    return node
}
```

### Find Minimum/Maximum

```pseudocode
function findMin(node) {
    if node == null {
        return null
    }

    while node.left != null {
        node = node.left
    }
    return node
}

function findMax(node) {
    if node == null {
        return null
    }

    while node.right != null {
        node = node.right
    }
    return node
}
```

### Inorder Successor/Predecessor

```pseudocode
function inorderSuccessor(root, node) {
    // If right subtree exists, successor is leftmost in right subtree
    if node.right != null {
        return findMin(node.right)
    }

    // Otherwise, go up to find first ancestor where node is in left subtree
    successor = null
    current = root

    while current != null {
        if node.data < current.data {
            successor = current
            current = current.left
        } else if node.data > current.data {
            current = current.right
        } else {
            break
        }
    }

    return successor
}

function inorderPredecessor(root, node) {
    // If left subtree exists, predecessor is rightmost in left subtree
    if node.left != null {
        return findMax(node.left)
    }

    // Otherwise, go up to find first ancestor where node is in right subtree
    predecessor = null
    current = root

    while current != null {
        if node.data > current.data {
            predecessor = current
            current = current.right
        } else if node.data < current.data {
            current = current.left
        } else {
            break
        }
    }

    return predecessor
}
```

### Validate BST

```pseudocode
function isValidBST(root) {
    return validate(root, MIN_VALUE, MAX_VALUE)
}

function validate(node, minVal, maxVal) {
    if node == null {
        return true
    }

    if node.data <= minVal or node.data >= maxVal {
        return false
    }

    return validate(node.left, minVal, node.data)
           and validate(node.right, node.data, maxVal)
}

// Alternative: Inorder traversal should be sorted
function isValidBSTInorder(root) {
    prev = null

    function inorder(node) {
        if node == null {
            return true
        }

        if !inorder(node.left) {
            return false
        }

        if prev != null and node.data <= prev.data {
            return false
        }
        prev = node

        return inorder(node.right)
    }

    return inorder(root)
}
```

## Implementation

```pseudocode
class BinarySearchTree {
    root = null
    size = 0

    function insert(key) {
        this.root = insertNode(this.root, key)
        this.size = this.size + 1
    }

    function insertNode(node, key) {
        if node == null {
            return new TreeNode(key)
        }

        if key < node.data {
            node.left = insertNode(node.left, key)
        } else if key > node.data {
            node.right = insertNode(node.right, key)
        }

        return node
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
        } else {
            return searchNode(node.right, key)
        }
    }

    function delete(key) {
        this.root = deleteNode(this.root, key)
        this.size = this.size - 1
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
            }
            if node.right == null {
                return node.left
            }

            minNode = findMinNode(node.right)
            node.data = minNode.data
            node.right = deleteNode(node.right, minNode.data)
        }

        return node
    }

    function findMin() {
        return findMinNode(this.root)
    }

    function findMinNode(node) {
        while node != null and node.left != null {
            node = node.left
        }
        return node
    }

    function findMax() {
        return findMaxNode(this.root)
    }

    function findMaxNode(node) {
        while node != null and node.right != null {
            node = node.right
        }
        return node
    }

    function contains(key) {
        return search(key) != null
    }

    function inorderTraversal() {
        result = []
        inorder(this.root, result)
        return result
    }

    function inorder(node, result) {
        if node != null {
            inorder(node.left, result)
            result.append(node.data)
            inorder(node.right, result)
        }
    }

    function isEmpty() {
        return this.root == null
    }

    function size() {
        return this.size
    }
}
```

## Classic Algorithms

### Range Query

```pseudocode
function rangeQuery(node, low, high, result) {
    if node == null {
        return
    }

    // If node is greater than low, check left subtree
    if node.data > low {
        rangeQuery(node.left, low, high, result)
    }

    // If node is in range, include it
    if node.data >= low and node.data <= high {
        result.append(node.data)
    }

    // If node is less than high, check right subtree
    if node.data < high {
        rangeQuery(node.right, low, high, result)
    }
}
```

### Kth Smallest Element

```pseudocode
function kthSmallest(root, k) {
    count = 0
    result = null

    function inorder(node) {
        if node == null or result != null {
            return
        }

        inorder(node.left)

        count = count + 1
        if count == k {
            result = node.data
            return
        }

        inorder(node.right)
    }

    inorder(root)
    return result
}
```

### Build BST from Sorted Array

```pseudocode
function sortedArrayToBST(array) {
    return buildBST(array, 0, array.length - 1)
}

function buildBST(array, left, right) {
    if left > right {
        return null
    }

    mid = left + (right - left) / 2
    node = new TreeNode(array[mid])

    node.left = buildBST(array, left, mid - 1)
    node.right = buildBST(array, mid + 1, right)

    return node
}
```

## Use Cases

- **Dictionaries**: Fast key-value lookup
- **Database indexing**: B-trees (generalized BST) for databases
- **Symbol tables**: Compilers storing identifiers
- **Priority queues**: Alternative to heaps
- **Sorted data maintenance**: Dynamic sorted collection
- **Range queries**: Find all elements in a range
- **Autocomplete**: Combined with tries

## Advantages

- **Efficient operations**: O(log n) average case
- **Sorted iteration**: Inorder traversal is sorted
- **Dynamic**: Insertions and deletions without rebuilding
- **Range queries**: Efficient range-based searches
- **Floor/ceiling**: Find closest elements efficiently

## Disadvantages

- **Can become unbalanced**: O(n) worst case
- **No guaranteed balance**: Requires self-balancing variant
- **Sensitive to insertion order**: Sorted insertions create degenerate tree
- **Memory overhead**: Two pointers per node
- **Complex deletion**: Three cases to handle

## Comparison with Alternatives

| Aspect          | BST          | Hash Table    | Sorted Array | AVL Tree     |
|-----------------|--------------|---------------|--------------|--------------|
| Search          | O(log n)*    | O(1)          | O(log n)     | O(log n)     |
| Insert          | O(log n)*    | O(1)          | O(n)         | O(log n)     |
| Delete          | O(log n)*    | O(1)          | O(n)         | O(log n)     |
| Ordered ops     | Yes          | No            | Yes          | Yes          |
| Range queries   | O(log n + k) | O(n)          | O(log n + k) | O(log n + k) |
| Min/Max         | O(log n)*    | O(n)          | O(1)         | O(log n)     |
| Balance         | Not guaranteed| N/A          | N/A          | Guaranteed   |

*Average case; worst case is O(n) for unbalanced BST

## Common Pitfalls

- **Not handling duplicates**: Decide on duplicate policy upfront
- **Sorted input**: Creates degenerate tree (use balanced variant)
- **Deletion complexity**: Three cases often confused
- **Forgetting to update size**: After insert/delete
- **Null pointer errors**: Not checking null before access
- **Iterator invalidation**: Modifying tree during traversal

## Related Structures

- **AVL Tree**: Self-balancing BST with height constraint
- **Red-Black Tree**: Self-balancing with color properties
- **Splay Tree**: Self-adjusting with amortized O(log n)
- **Treap**: BST + heap properties using random priorities
- **B-Tree**: Generalized BST for disk-based storage

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
