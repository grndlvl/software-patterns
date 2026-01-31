# Interval Tree

## Overview

An Interval Tree is an augmented self-balancing binary search tree designed to efficiently find all intervals that overlap with a given interval or point. It extends a standard BST by storing the maximum endpoint in each subtree, enabling O(log n) overlap queries. Interval trees are fundamental for computational geometry, scheduling, and any application involving range-based data.

## Properties

- **Augmented BST**: Each node stores an interval and max endpoint in subtree
- **Ordered by left endpoint**: BST ordering on interval start points
- **Max augmentation**: Each node stores maximum right endpoint in its subtree
- **Self-balancing**: Typically implemented with Red-Black or AVL trees
- **Overlap detection**: Efficiently finds all overlapping intervals

## Time Complexity

| Operation           | Time         | Notes                           |
|---------------------|--------------|--------------------------------|
| Insert              | O(log n)     | Standard BST insert + update max|
| Delete              | O(log n)     | Standard BST delete + update max|
| Search (exact)      | O(log n)     | Find specific interval          |
| Find overlapping    | O(log n + k) | k = number of overlapping intervals|
| Find any overlap    | O(log n)     | Return first overlapping interval|
| Query point         | O(log n + k) | All intervals containing point  |

## Space Complexity

O(n) for n intervals. Each node stores interval, max value, and standard BST pointers.

## Structure

### Interval Representation

```pseudocode
class Interval {
    low      // Left endpoint (inclusive)
    high     // Right endpoint (inclusive)
    data     // Associated data (optional)
}

// Overlap condition: a.low <= b.high AND b.low <= a.high
function overlaps(a, b) {
    return a.low <= b.high and b.low <= a.high
}
```

### Node Structure

```pseudocode
class IntervalNode {
    interval         // The stored interval
    max              // Maximum high value in subtree rooted here
    left = null      // Left child
    right = null     // Right child
    parent = null    // Parent (for balancing)
    // Additional fields for balancing (color for Red-Black, height for AVL)
}
```

## Operations

### Insert

```pseudocode
function insert(tree, interval) {
    node = new IntervalNode()
    node.interval = interval
    node.max = interval.high

    // Standard BST insert (ordered by interval.low)
    if tree.root == null {
        tree.root = node
        return node
    }

    current = tree.root
    while true {
        // Update max on the path down
        current.max = max(current.max, interval.high)

        if interval.low < current.interval.low {
            if current.left == null {
                current.left = node
                node.parent = current
                break
            }
            current = current.left
        } else {
            if current.right == null {
                current.right = node
                node.parent = current
                break
            }
            current = current.right
        }
    }

    // Rebalance if using Red-Black or AVL
    rebalance(tree, node)

    return node
}
```

### Update Max After Structural Changes

```pseudocode
function updateMax(node) {
    if node == null {
        return
    }

    node.max = node.interval.high

    if node.left != null {
        node.max = max(node.max, node.left.max)
    }
    if node.right != null {
        node.max = max(node.max, node.right.max)
    }
}

// Call after rotations during rebalancing
function updateMaxUpward(node) {
    while node != null {
        oldMax = node.max
        updateMax(node)
        if node.max == oldMax {
            break  // No change, stop propagating
        }
        node = node.parent
    }
}
```

### Find Any Overlapping Interval

```pseudocode
function findOverlap(tree, interval) {
    node = tree.root

    while node != null {
        // Check if current interval overlaps
        if overlaps(node.interval, interval) {
            return node.interval
        }

        // Decision: go left or right?
        // Go left if left subtree exists AND its max >= interval.low
        // (meaning there could be an overlapping interval in left subtree)
        if node.left != null and node.left.max >= interval.low {
            node = node.left
        } else {
            node = node.right
        }
    }

    return null  // No overlap found
}
```

### Find All Overlapping Intervals

```pseudocode
function findAllOverlaps(tree, interval) {
    result = []
    findAllOverlapsHelper(tree.root, interval, result)
    return result
}

function findAllOverlapsHelper(node, interval, result) {
    if node == null {
        return
    }

    // Check current node
    if overlaps(node.interval, interval) {
        result.append(node.interval)
    }

    // Search left subtree if there might be overlaps
    if node.left != null and node.left.max >= interval.low {
        findAllOverlapsHelper(node.left, interval, result)
    }

    // Search right subtree if there might be overlaps
    // Right subtree has intervals with low > node.interval.low
    // They can overlap if node.interval.low <= interval.high
    if node.right != null and node.interval.low <= interval.high {
        findAllOverlapsHelper(node.right, interval, result)
    }
}
```

### Find Intervals Containing a Point

```pseudocode
function findIntervalsContaining(tree, point) {
    interval = new Interval(point, point)
    return findAllOverlaps(tree, interval)
}
```

### Delete

```pseudocode
function delete(tree, interval) {
    // Find the node
    node = findNode(tree.root, interval)

    if node == null {
        return false
    }

    // Standard BST delete
    if node.left == null {
        transplant(tree, node, node.right)
        updateMaxUpward(node.parent)
    } else if node.right == null {
        transplant(tree, node, node.left)
        updateMaxUpward(node.parent)
    } else {
        // Find successor
        successor = minimum(node.right)

        if successor.parent != node {
            transplant(tree, successor, successor.right)
            successor.right = node.right
            successor.right.parent = successor
        }

        transplant(tree, node, successor)
        successor.left = node.left
        successor.left.parent = successor

        updateMaxUpward(successor)
    }

    // Rebalance and update max values
    rebalance(tree, node.parent)

    return true
}

function findNode(node, interval) {
    while node != null {
        if interval.low == node.interval.low and interval.high == node.interval.high {
            return node
        }
        if interval.low < node.interval.low {
            node = node.left
        } else {
            node = node.right
        }
    }
    return null
}
```

## Complete Implementation

```pseudocode
class IntervalTree {
    root = null

    function insert(low, high, data = null) {
        interval = new Interval(low, high, data)
        // ... insert implementation
    }

    function delete(low, high) {
        interval = new Interval(low, high)
        // ... delete implementation
    }

    function findOverlap(low, high) {
        interval = new Interval(low, high)
        // ... findOverlap implementation
    }

    function findAllOverlaps(low, high) {
        interval = new Interval(low, high)
        // ... findAllOverlaps implementation
    }

    function findContaining(point) {
        return findAllOverlaps(point, point)
    }

    function isEmpty() {
        return this.root == null
    }
}
```

## Use Cases

### Calendar/Scheduling

```pseudocode
class Calendar {
    events = new IntervalTree()

    function addEvent(startTime, endTime, eventData) {
        events.insert(startTime, endTime, eventData)
    }

    function getConflicts(startTime, endTime) {
        return events.findAllOverlaps(startTime, endTime)
    }

    function isTimeSlotFree(startTime, endTime) {
        return events.findOverlap(startTime, endTime) == null
    }

    function getEventsAt(time) {
        return events.findContaining(time)
    }
}
```

### Computational Geometry (Line Segment Intersection)

```pseudocode
function findIntersectingSegments(segments) {
    // Project segments onto x-axis as intervals
    tree = new IntervalTree()
    intersections = []

    for segment in segments {
        interval = (segment.x1, segment.x2)

        // Find all segments that might intersect in x-range
        candidates = tree.findAllOverlaps(interval)

        for candidate in candidates {
            if actuallyIntersects(segment, candidate.data) {
                intersections.append((segment, candidate.data))
            }
        }

        tree.insert(interval.low, interval.high, segment)
    }

    return intersections
}
```

### Database Range Queries

```pseudocode
class RangeIndex {
    tree = new IntervalTree()

    function addRange(low, high, recordIds) {
        tree.insert(low, high, recordIds)
    }

    function queryRange(low, high) {
        overlapping = tree.findAllOverlaps(low, high)

        result = new Set()
        for interval in overlapping {
            result.addAll(interval.data)
        }

        return result
    }
}
```

### Network Packet Classification

```pseudocode
class PacketClassifier {
    portRanges = new IntervalTree()

    function addRule(portLow, portHigh, action) {
        portRanges.insert(portLow, portHigh, action)
    }

    function classify(port) {
        matches = portRanges.findContaining(port)
        // Apply priority rules to matches
        return selectHighestPriority(matches)
    }
}
```

## Advantages

- **Efficient overlap queries**: O(log n + k) for finding all overlaps
- **Standard BST base**: Well-understood algorithms
- **Dynamic**: Supports insertions and deletions
- **Versatile**: Works for various interval-based problems
- **Space efficient**: O(n) space with minimal overhead

## Disadvantages

- **One-dimensional**: Only handles 1D intervals directly
- **Complex balancing**: Must update max during rotations
- **No bulk operations**: Must insert intervals one by one
- **Assumes closed intervals**: May need modification for open intervals
- **Point queries need all k**: Cannot stop early when finding containment

## Comparison with Alternatives

| Aspect              | Interval Tree | Segment Tree | Range Tree    |
|---------------------|---------------|--------------|---------------|
| Space               | O(n)          | O(n)         | O(n log n)    |
| Insert              | O(log n)      | O(log n)     | O(log² n)     |
| Delete              | O(log n)      | Complex      | O(log² n)     |
| Overlap query       | O(log n + k)  | O(log n + k) | N/A           |
| Point query         | O(log n + k)  | O(log n + k) | O(log n + k)  |
| Dynamic             | Yes           | Limited      | Yes           |
| Best for            | Intervals     | Fixed ranges | Points        |

## Variants

### Centered Interval Tree

```pseudocode
// Alternative structure: intervals stored at nodes based on center point
class CenteredIntervalTree {
    center           // Center point
    left             // Intervals entirely left of center
    right            // Intervals entirely right of center
    // Intervals crossing center, sorted by:
    sortedByLow[]    // left endpoints
    sortedByHigh[]   // right endpoints
}

// Query time: O(log n + k)
// Good for static sets of intervals
```

### Augmented Interval Tree (with count)

```pseudocode
class CountingIntervalTree {
    // Standard interval tree plus:
    count            // Number of intervals in subtree

    function countOverlaps(interval) {
        // Return count without enumerating
    }
}
```

## Common Pitfalls

- **Forgetting to update max**: After insert, delete, or rotation
- **Wrong overlap condition**: Must check both endpoints
- **Not handling duplicates**: Same interval inserted twice
- **Incorrect search pruning**: May miss overlaps if pruning wrong subtree
- **Off-by-one with open/closed intervals**: Be consistent
- **Not rebalancing**: Tree can degenerate to O(n) operations
- **Comparing with point vs interval**: Different query types

## Related Structures

- **Segment Tree**: For aggregate queries over ranges
- **Range Tree**: For multi-dimensional point queries
- **R-Tree**: For spatial data and rectangles
- **KD-Tree**: For multi-dimensional point data
- **Fenwick Tree**: For prefix sums and ranges

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
