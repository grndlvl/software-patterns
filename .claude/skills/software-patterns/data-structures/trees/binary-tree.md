# Binary Tree

## Overview

A binary tree is a hierarchical data structure where each node has at most two children, referred to as the left child and right child. It forms the foundation for many important data structures including binary search trees, heaps, and expression trees.

## Properties

- **At most two children**: Each node has 0, 1, or 2 children
- **Recursive structure**: Each subtree is itself a binary tree
- **Root**: Single entry point at the top
- **Leaves**: Nodes with no children
- **Height**: Longest path from root to leaf

### Tree Metrics

- **Height of node**: Longest path from node to a leaf
- **Depth of node**: Distance from root to node
- **Height of tree**: Height of root node
- **Level**: All nodes at same depth

### Special Binary Trees

- **Full**: Every node has 0 or 2 children
- **Complete**: All levels filled except possibly last, filled left to right
- **Perfect**: All internal nodes have 2 children, all leaves at same level
- **Balanced**: Height is O(log n)
- **Degenerate**: Each node has at most 1 child (essentially a linked list)

## Time Complexity

| Operation     | Average     | Worst (Degenerate) |
|---------------|-------------|---------------------|
| Access        | O(n)        | O(n)                |
| Search        | O(n)        | O(n)                |
| Insert        | O(n)        | O(n)                |
| Delete        | O(n)        | O(n)                |
| Traversal     | O(n)        | O(n)                |

*Note: For BST or balanced trees, average operations are O(log n)*

## Space Complexity

O(n) for n nodes. Additional O(h) space for recursive traversals where h is height.

For a complete binary tree with n nodes: height h = floor(log2(n))

## Operations

### Node Structure

```pseudocode
class TreeNode {
    data
    left = null
    right = null
    parent = null  // Optional, useful for some algorithms

    function constructor(data) {
        this.data = data
    }

    function isLeaf() {
        return this.left == null and this.right == null
    }
}
```

### Tree Traversals

#### Preorder (Root-Left-Right)

```pseudocode
function preorder(node) {
    if node == null {
        return
    }

    process(node.data)      // Visit root first
    preorder(node.left)     // Then left subtree
    preorder(node.right)    // Then right subtree
}

// Iterative version
function preorderIterative(root) {
    if root == null {
        return
    }

    stack = new Stack()
    stack.push(root)

    while !stack.isEmpty() {
        node = stack.pop()
        process(node.data)

        // Push right first so left is processed first
        if node.right != null {
            stack.push(node.right)
        }
        if node.left != null {
            stack.push(node.left)
        }
    }
}
```

#### Inorder (Left-Root-Right)

```pseudocode
function inorder(node) {
    if node == null {
        return
    }

    inorder(node.left)      // Visit left subtree first
    process(node.data)      // Then root
    inorder(node.right)     // Then right subtree
}

// Iterative version
function inorderIterative(root) {
    stack = new Stack()
    current = root

    while current != null or !stack.isEmpty() {
        // Go to leftmost node
        while current != null {
            stack.push(current)
            current = current.left
        }

        current = stack.pop()
        process(current.data)
        current = current.right
    }
}
```

#### Postorder (Left-Right-Root)

```pseudocode
function postorder(node) {
    if node == null {
        return
    }

    postorder(node.left)    // Visit left subtree first
    postorder(node.right)   // Then right subtree
    postorder(node.data)    // Then root
}

// Iterative version (using two stacks)
function postorderIterative(root) {
    if root == null {
        return
    }

    stack1 = new Stack()
    stack2 = new Stack()

    stack1.push(root)

    while !stack1.isEmpty() {
        node = stack1.pop()
        stack2.push(node)

        if node.left != null {
            stack1.push(node.left)
        }
        if node.right != null {
            stack1.push(node.right)
        }
    }

    while !stack2.isEmpty() {
        process(stack2.pop().data)
    }
}
```

#### Level Order (Breadth-First)

```pseudocode
function levelOrder(root) {
    if root == null {
        return
    }

    queue = new Queue()
    queue.enqueue(root)

    while !queue.isEmpty() {
        node = queue.dequeue()
        process(node.data)

        if node.left != null {
            queue.enqueue(node.left)
        }
        if node.right != null {
            queue.enqueue(node.right)
        }
    }
}

// Level by level with level markers
function levelOrderByLevel(root) {
    if root == null {
        return []
    }

    result = []
    queue = new Queue()
    queue.enqueue(root)

    while !queue.isEmpty() {
        levelSize = queue.size()
        currentLevel = []

        for i from 0 to levelSize - 1 {
            node = queue.dequeue()
            currentLevel.append(node.data)

            if node.left != null {
                queue.enqueue(node.left)
            }
            if node.right != null {
                queue.enqueue(node.right)
            }
        }

        result.append(currentLevel)
    }

    return result
}
```

### Height Calculation

```pseudocode
function height(node) {
    if node == null {
        return -1  // Or 0, depending on convention
    }

    leftHeight = height(node.left)
    rightHeight = height(node.right)

    return 1 + max(leftHeight, rightHeight)
}
```

### Count Nodes

```pseudocode
function countNodes(node) {
    if node == null {
        return 0
    }

    return 1 + countNodes(node.left) + countNodes(node.right)
}
```

### Check if Balanced

```pseudocode
function isBalanced(node) {
    return checkBalance(node) != -1
}

function checkBalance(node) {
    if node == null {
        return 0
    }

    leftHeight = checkBalance(node.left)
    if leftHeight == -1 {
        return -1
    }

    rightHeight = checkBalance(node.right)
    if rightHeight == -1 {
        return -1
    }

    if abs(leftHeight - rightHeight) > 1 {
        return -1
    }

    return 1 + max(leftHeight, rightHeight)
}
```

## Implementation

```pseudocode
class BinaryTree {
    root = null

    function insert(data) {
        newNode = new TreeNode(data)

        if this.root == null {
            this.root = newNode
            return
        }

        // Level-order insertion (complete tree)
        queue = new Queue()
        queue.enqueue(this.root)

        while !queue.isEmpty() {
            node = queue.dequeue()

            if node.left == null {
                node.left = newNode
                return
            } else {
                queue.enqueue(node.left)
            }

            if node.right == null {
                node.right = newNode
                return
            } else {
                queue.enqueue(node.right)
            }
        }
    }

    function find(data) {
        return findNode(this.root, data)
    }

    function findNode(node, data) {
        if node == null {
            return null
        }

        if node.data == data {
            return node
        }

        leftResult = findNode(node.left, data)
        if leftResult != null {
            return leftResult
        }

        return findNode(node.right, data)
    }

    function height() {
        return calculateHeight(this.root)
    }

    function calculateHeight(node) {
        if node == null {
            return -1
        }
        return 1 + max(calculateHeight(node.left), calculateHeight(node.right))
    }

    function size() {
        return countNodes(this.root)
    }

    function countNodes(node) {
        if node == null {
            return 0
        }
        return 1 + countNodes(node.left) + countNodes(node.right)
    }

    function isEmpty() {
        return this.root == null
    }
}
```

## Classic Algorithms

### Lowest Common Ancestor

```pseudocode
function lowestCommonAncestor(root, p, q) {
    if root == null or root == p or root == q {
        return root
    }

    left = lowestCommonAncestor(root.left, p, q)
    right = lowestCommonAncestor(root.right, p, q)

    if left != null and right != null {
        return root  // p and q are on different sides
    }

    return left != null ? left : right
}
```

### Mirror/Invert Tree

```pseudocode
function invertTree(node) {
    if node == null {
        return null
    }

    // Swap children
    temp = node.left
    node.left = node.right
    node.right = temp

    // Recursively invert subtrees
    invertTree(node.left)
    invertTree(node.right)

    return node
}
```

### Check Symmetric

```pseudocode
function isSymmetric(root) {
    if root == null {
        return true
    }
    return isMirror(root.left, root.right)
}

function isMirror(node1, node2) {
    if node1 == null and node2 == null {
        return true
    }
    if node1 == null or node2 == null {
        return false
    }
    return node1.data == node2.data
           and isMirror(node1.left, node2.right)
           and isMirror(node1.right, node2.left)
}
```

## Use Cases

- **Expression trees**: Represent mathematical expressions
- **Decision trees**: Machine learning, game AI
- **Huffman coding**: Data compression
- **File systems**: Directory structure (n-ary variant)
- **DOM**: HTML document object model
- **Syntax trees**: Compilers, parsers
- **Foundation for BST, heap**: Building block for specialized trees

## Advantages

- **Hierarchical representation**: Natural for parent-child relationships
- **Efficient operations**: O(log n) for balanced trees
- **Flexible structure**: Many specialized variants
- **Recursive algorithms**: Clean, elegant solutions
- **Ordered traversals**: Inorder gives sorted sequence for BST

## Disadvantages

- **Can become unbalanced**: Degenerates to O(n) operations
- **Pointer overhead**: Two pointers per node minimum
- **No random access**: Must traverse to find elements
- **Complex deletion**: Especially with two children
- **Memory locality**: Nodes scattered in memory

## Comparison with Alternatives

| Aspect           | Binary Tree | N-ary Tree | Linked List | Array    |
|------------------|-------------|------------|-------------|----------|
| Children/node    | 2           | N          | 1           | N/A      |
| Search           | O(n)        | O(n)       | O(n)        | O(n)/O(1)|
| Hierarchical     | Yes         | Yes        | No          | No       |
| Memory/node      | 2 pointers  | N pointers | 1 pointer   | None     |
| Balanced height  | O(log n)    | O(logN n)  | O(n)        | N/A      |

## Common Pitfalls

- **Null checks**: Always check for null before accessing children
- **Stack overflow**: Deep recursion on large/unbalanced trees
- **Off-by-one in height**: Clarify if empty tree height is -1 or 0
- **Forgetting base case**: Recursion must handle null nodes
- **Modifying during traversal**: Can cause unexpected behavior
- **Memory leaks**: Not cleaning up deleted nodes properly

## Related Structures

- **Binary Search Tree**: Binary tree with ordering property
- **AVL Tree**: Self-balancing BST
- **Red-Black Tree**: Self-balancing with color properties
- **Heap**: Complete binary tree with heap property
- **Trie**: Tree for string prefix matching
- **B-Tree**: Balanced tree with multiple keys per node

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
