# K-D Tree

## Overview

A K-D Tree (K-Dimensional Tree) is a space-partitioning data structure for organizing points in k-dimensional space. It's a binary tree where each node represents a point and partitions the space along one dimension. K-D trees enable efficient range searches and nearest neighbor queries, making them fundamental for computational geometry, machine learning (k-NN), and spatial databases.

## Properties

- **Binary tree**: Each node has at most two children
- **Splitting dimension**: Each level splits on a different dimension (cycling through)
- **Space partitioning**: Divides space into nested half-spaces
- **Point storage**: Each node stores one k-dimensional point
- **Balanced construction**: Can be built balanced in O(n log n)

## Time Complexity

| Operation              | Average     | Worst Case | Notes                        |
|------------------------|-------------|------------|------------------------------|
| Build (balanced)       | O(n log n)  | O(n log n) | Using median selection       |
| Insert                 | O(log n)    | O(n)       | Can unbalance tree           |
| Delete                 | O(log n)    | O(n)       | Complex, may need rebuild    |
| Nearest Neighbor       | O(log n)    | O(n)       | Depends on distribution      |
| Range Search           | O(√n + k)   | O(n)       | k = points in range          |

Note: For high dimensions, performance degrades toward O(n) due to "curse of dimensionality."

## Space Complexity

O(n) for n points. Each node stores the point and two child pointers.

## Structure

### Node Structure

```pseudocode
class KDNode {
    point[]         // k-dimensional point
    splitDimension  // Which dimension this node splits on
    left = null     // Points with smaller value in split dimension
    right = null    // Points with greater or equal value
}

class KDTree {
    root = null
    k              // Number of dimensions
}
```

## Operations

### Build Tree (Balanced)

```pseudocode
function buildKDTree(points, k) {
    tree = new KDTree()
    tree.k = k
    tree.root = buildHelper(points, 0, k)
    return tree
}

function buildHelper(points, depth, k) {
    if points.isEmpty() {
        return null
    }

    // Select dimension to split on (cycle through dimensions)
    dimension = depth mod k

    // Sort by current dimension and find median
    sort(points by point[dimension])
    medianIndex = length(points) / 2

    // Create node with median point
    node = new KDNode()
    node.point = points[medianIndex]
    node.splitDimension = dimension

    // Recursively build subtrees
    node.left = buildHelper(points[0..medianIndex-1], depth + 1, k)
    node.right = buildHelper(points[medianIndex+1..], depth + 1, k)

    return node
}

// Time: O(n log n) with O(n) median finding per level
// Better: O(n log n) using linear-time median selection
```

### Insert

```pseudocode
function insert(tree, point) {
    tree.root = insertHelper(tree.root, point, 0, tree.k)
}

function insertHelper(node, point, depth, k) {
    if node == null {
        newNode = new KDNode()
        newNode.point = point
        newNode.splitDimension = depth mod k
        return newNode
    }

    dimension = node.splitDimension

    if point[dimension] < node.point[dimension] {
        node.left = insertHelper(node.left, point, depth + 1, k)
    } else {
        node.right = insertHelper(node.right, point, depth + 1, k)
    }

    return node
}
```

### Search (Exact Point)

```pseudocode
function search(tree, point) {
    return searchHelper(tree.root, point, 0, tree.k)
}

function searchHelper(node, point, depth, k) {
    if node == null {
        return false
    }

    if pointsEqual(node.point, point) {
        return true
    }

    dimension = depth mod k

    if point[dimension] < node.point[dimension] {
        return searchHelper(node.left, point, depth + 1, k)
    } else {
        return searchHelper(node.right, point, depth + 1, k)
    }
}
```

### Nearest Neighbor Search

```pseudocode
function nearestNeighbor(tree, target) {
    best = new BestHolder()
    best.node = null
    best.distance = infinity

    nearestHelper(tree.root, target, 0, tree.k, best)

    return best.node.point
}

function nearestHelper(node, target, depth, k, best) {
    if node == null {
        return
    }

    // Check current node
    dist = distance(node.point, target)
    if dist < best.distance {
        best.distance = dist
        best.node = node
    }

    dimension = depth mod k
    diff = target[dimension] - node.point[dimension]

    // Search the side that contains target first
    if diff < 0 {
        first = node.left
        second = node.right
    } else {
        first = node.right
        second = node.left
    }

    nearestHelper(first, target, depth + 1, k, best)

    // Only search other side if it could contain closer point
    if abs(diff) < best.distance {
        nearestHelper(second, target, depth + 1, k, best)
    }
}

function distance(p1, p2) {
    sum = 0
    for i from 0 to k - 1 {
        sum += (p1[i] - p2[i])^2
    }
    return sqrt(sum)
}
```

### K Nearest Neighbors

```pseudocode
function kNearestNeighbors(tree, target, k) {
    // Use max-heap to track k nearest points
    heap = new MaxHeap()  // Ordered by distance (max at top)

    kNearestHelper(tree.root, target, 0, tree.k, k, heap)

    // Extract results
    result = []
    while not heap.isEmpty() {
        result.prepend(heap.extractMax().point)
    }
    return result
}

function kNearestHelper(node, target, depth, dimensions, k, heap) {
    if node == null {
        return
    }

    dist = distance(node.point, target)

    // Add to heap if it's one of k closest
    if heap.size() < k {
        heap.insert((dist, node))
    } else if dist < heap.peekMax().distance {
        heap.extractMax()
        heap.insert((dist, node))
    }

    dimension = depth mod dimensions
    diff = target[dimension] - node.point[dimension]

    if diff < 0 {
        first = node.left
        second = node.right
    } else {
        first = node.right
        second = node.left
    }

    kNearestHelper(first, target, depth + 1, dimensions, k, heap)

    // Prune if no point in other subtree could be closer
    if heap.size() < k or abs(diff) < heap.peekMax().distance {
        kNearestHelper(second, target, depth + 1, dimensions, k, heap)
    }
}
```

### Range Search

```pseudocode
function rangeSearch(tree, lowerBound, upperBound) {
    // Find all points within hyper-rectangle [lowerBound, upperBound]
    result = []
    rangeHelper(tree.root, lowerBound, upperBound, 0, tree.k, result)
    return result
}

function rangeHelper(node, lower, upper, depth, k, result) {
    if node == null {
        return
    }

    // Check if current point is in range
    if pointInRange(node.point, lower, upper) {
        result.append(node.point)
    }

    dimension = depth mod k

    // Check if we need to search left subtree
    if lower[dimension] <= node.point[dimension] {
        rangeHelper(node.left, lower, upper, depth + 1, k, result)
    }

    // Check if we need to search right subtree
    if upper[dimension] >= node.point[dimension] {
        rangeHelper(node.right, lower, upper, depth + 1, k, result)
    }
}

function pointInRange(point, lower, upper) {
    for i from 0 to k - 1 {
        if point[i] < lower[i] or point[i] > upper[i] {
            return false
        }
    }
    return true
}
```

### Delete

```pseudocode
function delete(tree, point) {
    tree.root = deleteHelper(tree.root, point, 0, tree.k)
}

function deleteHelper(node, point, depth, k) {
    if node == null {
        return null
    }

    dimension = depth mod k

    if pointsEqual(node.point, point) {
        // Found the node to delete
        if node.right != null {
            // Replace with min in right subtree (in split dimension)
            minNode = findMin(node.right, dimension, depth + 1, k)
            node.point = minNode.point
            node.right = deleteHelper(node.right, minNode.point, depth + 1, k)
        } else if node.left != null {
            // Replace with min in left subtree, move left to right
            minNode = findMin(node.left, dimension, depth + 1, k)
            node.point = minNode.point
            node.right = node.left
            node.left = null
            node.right = deleteHelper(node.right, minNode.point, depth + 1, k)
        } else {
            // Leaf node
            return null
        }
    } else if point[dimension] < node.point[dimension] {
        node.left = deleteHelper(node.left, point, depth + 1, k)
    } else {
        node.right = deleteHelper(node.right, point, depth + 1, k)
    }

    return node
}

function findMin(node, dimension, depth, k) {
    if node == null {
        return null
    }

    currentDim = depth mod k

    if currentDim == dimension {
        // Min is in left subtree
        if node.left == null {
            return node
        }
        return findMin(node.left, dimension, depth + 1, k)
    }

    // Min could be anywhere, search both subtrees
    leftMin = findMin(node.left, dimension, depth + 1, k)
    rightMin = findMin(node.right, dimension, depth + 1, k)

    return minInDimension(node, leftMin, rightMin, dimension)
}
```

## Use Cases

### K-Nearest Neighbors Classification

```pseudocode
class KNNClassifier {
    tree
    labels = new Map()  // point -> label

    function train(points, labels) {
        tree = buildKDTree(points, k)
        for i from 0 to length(points) - 1 {
            this.labels[points[i]] = labels[i]
        }
    }

    function predict(point, k) {
        neighbors = kNearestNeighbors(tree, point, k)

        // Majority vote
        votes = new Map()
        for neighbor in neighbors {
            label = labels[neighbor]
            votes[label] = votes.get(label, 0) + 1
        }

        return argmax(votes)
    }
}
```

### Collision Detection

```pseudocode
function findPotentialCollisions(objects, radius) {
    // Build k-d tree of object centers
    points = [obj.center for obj in objects]
    tree = buildKDTree(points, 3)

    collisions = []

    for obj in objects {
        // Range search for objects within collision distance
        lower = [obj.center[i] - 2*radius for i in 0..2]
        upper = [obj.center[i] + 2*radius for i in 0..2]

        nearby = rangeSearch(tree, lower, upper)

        for other in nearby {
            if obj != other and distance(obj.center, other.center) < 2*radius {
                collisions.append((obj, other))
            }
        }
    }

    return unique(collisions)
}
```

### Database Spatial Index

```pseudocode
class SpatialIndex {
    tree

    function buildIndex(records) {
        points = [record.location for record in records]
        tree = buildKDTree(points, 2)
    }

    function findNearby(location, radius) {
        lower = [location[0] - radius, location[1] - radius]
        upper = [location[0] + radius, location[1] + radius]

        candidates = rangeSearch(tree, lower, upper)

        // Filter to actual circle
        return [p for p in candidates if distance(p, location) <= radius]
    }

    function findNearest(location) {
        return nearestNeighbor(tree, location)
    }
}
```

## Advantages

- **Efficient spatial queries**: O(log n) average for nearest neighbor
- **Generalizes to k dimensions**: Works for any dimensionality
- **Simple concept**: Easy to understand and implement
- **No preprocessing for queries**: Unlike grid-based methods
- **Handles non-uniform distributions**: Better than fixed grids

## Disadvantages

- **Curse of dimensionality**: Performance degrades for k > ~20
- **Unbalanced with insertions**: May need periodic rebalancing
- **Complex deletion**: Maintaining tree structure is tricky
- **Poor for high dimensions**: Linear scan may be faster
- **Static structure**: Best built once, not frequently modified

## Comparison with Alternatives

| Aspect           | K-D Tree    | R-Tree      | Quad/Octree | Ball Tree   |
|------------------|-------------|-------------|-------------|-------------|
| Dimensions       | Any         | 2-3         | 2-3         | Any         |
| Point queries    | O(log n)    | O(log n)    | O(log n)    | O(log n)    |
| Range queries    | O(√n + k)   | O(log n + k)| O(√n + k)   | O(√n + k)   |
| Insert           | O(log n)    | O(log n)    | O(log n)    | O(log n)    |
| Bulk build       | O(n log n)  | O(n log n)  | O(n)        | O(n log n)  |
| High dimensions  | Poor        | Poor        | N/A         | Better      |

## Common Pitfalls

- **Not cycling dimensions**: Must rotate through all k dimensions
- **Wrong distance metric**: Ensure consistent distance function
- **Not pruning properly**: Must check if other subtree could be closer
- **High-dimensional data**: Consider dimensionality reduction first
- **Unbalanced tree**: Random insertions can degrade performance
- **Ignoring ties**: Handle points with equal coordinates in split dimension

## Related Structures

- **R-Tree**: Better for rectangles and bulk operations
- **Ball Tree**: Better for high dimensions
- **Quad/Octree**: Fixed partitioning for 2D/3D
- **VP-Tree**: Vantage-point tree for metric spaces
- **Cover Tree**: For well-separated point sets
- **Locality-Sensitive Hashing**: For approximate nearest neighbor in high dimensions

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
