# Skip List

## Overview

A skip list is a probabilistic data structure that allows O(log n) search, insertion, and deletion within an ordered sequence. It achieves this by maintaining multiple levels of linked lists where higher levels "skip" over elements, similar to express lanes on a highway. Skip lists are simpler to implement than balanced trees while providing comparable performance.

## Properties

- **Probabilistic balancing**: Uses randomization instead of strict balancing rules
- **Multiple levels**: Each element may appear in multiple layers
- **Ordered**: Elements maintained in sorted order
- **Linked structure**: Uses forward pointers at each level
- **Expected O(log n)**: High probability of logarithmic operations

### Level Distribution

Each element has a level determined by coin flips:
- Level 1: All elements (100%)
- Level 2: ~50% of elements
- Level 3: ~25% of elements
- Level k: ~(1/2)^(k-1) of elements

## Time Complexity

| Operation | Average     | Worst       |
|-----------|-------------|-------------|
| Search    | O(log n)    | O(n)        |
| Insert    | O(log n)    | O(n)        |
| Delete    | O(log n)    | O(n)        |
| Min       | O(1)        | O(1)        |
| Max       | O(log n)    | O(n)        |

*Worst case is theoretically possible but extremely unlikely*

## Space Complexity

O(n) expected. Each element has on average 2 pointers (geometric series: 1 + 1/2 + 1/4 + ... = 2).

## Operations

### Node Structure

```pseudocode
class SkipListNode {
    value
    forward[]   // Array of forward pointers, one per level

    function constructor(value, level) {
        this.value = value
        this.forward = new Array(level + 1).fill(null)
    }
}
```

### Random Level Generation

```pseudocode
function randomLevel(maxLevel, probability = 0.5) {
    level = 0
    while random() < probability and level < maxLevel {
        level = level + 1
    }
    return level
}
```

### Search

```pseudocode
function search(value) {
    current = this.header

    // Start from highest level, move down
    for level from this.currentLevel down to 0 {
        while current.forward[level] != null
              and current.forward[level].value < value {
            current = current.forward[level]
        }
    }

    // Move to potential match
    current = current.forward[0]

    if current != null and current.value == value {
        return current
    }

    return null
}
```

### Insert

```pseudocode
function insert(value) {
    update = new Array(this.maxLevel + 1)
    current = this.header

    // Find position and track update points
    for level from this.currentLevel down to 0 {
        while current.forward[level] != null
              and current.forward[level].value < value {
            current = current.forward[level]
        }
        update[level] = current
    }

    current = current.forward[0]

    // If value already exists, update or skip
    if current != null and current.value == value {
        return false  // Duplicate
    }

    // Generate random level for new node
    newLevel = randomLevel(this.maxLevel)

    // If new level is higher than current, update header
    if newLevel > this.currentLevel {
        for level from this.currentLevel + 1 to newLevel {
            update[level] = this.header
        }
        this.currentLevel = newLevel
    }

    // Create new node
    newNode = new SkipListNode(value, newLevel)

    // Insert node by updating pointers
    for level from 0 to newLevel {
        newNode.forward[level] = update[level].forward[level]
        update[level].forward[level] = newNode
    }

    this.size = this.size + 1
    return true
}
```

### Delete

```pseudocode
function delete(value) {
    update = new Array(this.maxLevel + 1)
    current = this.header

    // Find node and track update points
    for level from this.currentLevel down to 0 {
        while current.forward[level] != null
              and current.forward[level].value < value {
            current = current.forward[level]
        }
        update[level] = current
    }

    current = current.forward[0]

    // If found, remove it
    if current != null and current.value == value {
        for level from 0 to this.currentLevel {
            if update[level].forward[level] != current {
                break
            }
            update[level].forward[level] = current.forward[level]
        }

        // Update current level if necessary
        while this.currentLevel > 0
              and this.header.forward[this.currentLevel] == null {
            this.currentLevel = this.currentLevel - 1
        }

        this.size = this.size - 1
        return true
    }

    return false
}
```

## Implementation

```pseudocode
class SkipList {
    maxLevel            // Maximum number of levels
    currentLevel = 0    // Current highest level in use
    header              // Header node (sentinel)
    size = 0
    probability = 0.5   // Probability of level increase

    function constructor(maxLevel = 16) {
        this.maxLevel = maxLevel
        this.header = new SkipListNode(null, maxLevel)
    }

    function randomLevel() {
        level = 0
        while random() < this.probability and level < this.maxLevel {
            level = level + 1
        }
        return level
    }

    function search(value) {
        current = this.header

        for level from this.currentLevel down to 0 {
            while current.forward[level] != null
                  and current.forward[level].value < value {
                current = current.forward[level]
            }
        }

        current = current.forward[0]

        if current != null and current.value == value {
            return current.value
        }
        return null
    }

    function insert(value) {
        update = new Array(this.maxLevel + 1)
        current = this.header

        for level from this.currentLevel down to 0 {
            while current.forward[level] != null
                  and current.forward[level].value < value {
                current = current.forward[level]
            }
            update[level] = current
        }

        current = current.forward[0]

        if current == null or current.value != value {
            newLevel = randomLevel()

            if newLevel > this.currentLevel {
                for level from this.currentLevel + 1 to newLevel {
                    update[level] = this.header
                }
                this.currentLevel = newLevel
            }

            newNode = new SkipListNode(value, newLevel)

            for level from 0 to newLevel {
                newNode.forward[level] = update[level].forward[level]
                update[level].forward[level] = newNode
            }

            this.size = this.size + 1
            return true
        }

        return false
    }

    function delete(value) {
        update = new Array(this.maxLevel + 1)
        current = this.header

        for level from this.currentLevel down to 0 {
            while current.forward[level] != null
                  and current.forward[level].value < value {
                current = current.forward[level]
            }
            update[level] = current
        }

        current = current.forward[0]

        if current != null and current.value == value {
            for level from 0 to this.currentLevel {
                if update[level].forward[level] != current {
                    break
                }
                update[level].forward[level] = current.forward[level]
            }

            while this.currentLevel > 0
                  and this.header.forward[this.currentLevel] == null {
                this.currentLevel = this.currentLevel - 1
            }

            this.size = this.size - 1
            return true
        }

        return false
    }

    function contains(value) {
        return search(value) != null
    }

    function min() {
        if this.header.forward[0] != null {
            return this.header.forward[0].value
        }
        return null
    }

    function isEmpty() {
        return this.size == 0
    }

    function toArray() {
        result = []
        current = this.header.forward[0]
        while current != null {
            result.append(current.value)
            current = current.forward[0]
        }
        return result
    }

    // Range query: find all values in [low, high]
    function range(low, high) {
        result = []
        current = this.header

        // Find starting point
        for level from this.currentLevel down to 0 {
            while current.forward[level] != null
                  and current.forward[level].value < low {
                current = current.forward[level]
            }
        }

        current = current.forward[0]

        // Collect values in range
        while current != null and current.value <= high {
            result.append(current.value)
            current = current.forward[0]
        }

        return result
    }

    function print() {
        for level from this.currentLevel down to 0 {
            output("Level " + level + ": ")
            current = this.header.forward[level]
            values = []
            while current != null {
                values.append(current.value)
                current = current.forward[level]
            }
            output(values.join(" -> "))
        }
    }
}
```

## Visual Representation

```
Level 3: Header -----------------------> 50 -----------------------> null
Level 2: Header --------> 20 ---------> 50 ---------> 80 ---------> null
Level 1: Header -> 10 -> 20 -> 30 -> 50 -> 60 -> 80 -> 90 -> null
Level 0: Header -> 10 -> 20 -> 30 -> 40 -> 50 -> 60 -> 70 -> 80 -> 90 -> null
```

## Use Cases

- **In-memory databases**: Redis sorted sets
- **Concurrent data structures**: Lock-free skip lists
- **Range queries**: Efficient interval searches
- **Priority queues**: Alternative to heaps
- **Indexing**: Simpler alternative to B-trees
- **LevelDB/RocksDB**: Memtable implementation

## Advantages

- **Simple implementation**: Much easier than balanced trees
- **Efficient**: O(log n) expected for all operations
- **Lock-free possible**: Easy to make concurrent
- **Range queries**: Natural support for ordered iteration
- **No rotations**: Unlike AVL/Red-Black trees
- **Cache-friendly**: Sequential access at each level

## Disadvantages

- **Probabilistic**: Worst case theoretically O(n)
- **More memory**: ~2n pointers vs n for linked list
- **Not deterministic**: Performance varies slightly
- **Reverse iteration**: Requires doubly-linked variant
- **Random number dependency**: Quality of random affects performance

## Comparison with Alternatives

| Aspect          | Skip List   | BST         | AVL/RB Tree | Hash Table  |
|-----------------|-------------|-------------|-------------|-------------|
| Search          | O(log n)*   | O(log n)    | O(log n)    | O(1)        |
| Insert          | O(log n)*   | O(log n)    | O(log n)    | O(1)        |
| Delete          | O(log n)*   | O(log n)    | O(log n)    | O(1)        |
| Ordered         | Yes         | Yes         | Yes         | No          |
| Range queries   | O(log n+k)  | O(log n+k)  | O(log n+k)  | O(n)        |
| Implementation  | Simple      | Simple      | Complex     | Simple      |
| Concurrent      | Easy        | Hard        | Hard        | Medium      |
| Deterministic   | No          | No*         | Yes         | No*         |

*Expected/average case

## Common Pitfalls

- **Poor random generator**: Use good PRNG for level generation
- **Max level too low**: Set based on expected n (maxLevel ≈ log2(n))
- **Probability choice**: 0.5 is optimal; 0.25 uses less space
- **Forgetting header updates**: When inserting higher-level nodes
- **Level tracking**: Must maintain current highest level
- **Null pointer errors**: Many pointer manipulations to get right

## Related Structures

- **Balanced BST**: Deterministic alternative (AVL, Red-Black)
- **B-Tree**: Better for disk-based storage
- **Treap**: Randomized BST with similar properties
- **Concurrent Skip List**: Thread-safe variant
- **Indexable Skip List**: Supports rank operations

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
