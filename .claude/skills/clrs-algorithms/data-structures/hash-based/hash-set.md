# Hash Set

## Overview

A hash set is an unordered collection of unique elements that uses hashing for O(1) average-time membership testing, insertion, and deletion. It's essentially a hash table where only keys are stored (or keys map to a dummy value). Hash sets are fundamental for deduplication, membership checks, and set operations.

## Properties

- **Unique elements**: No duplicates allowed
- **Unordered**: No guaranteed iteration order
- **Hash-based**: Uses hash function for fast operations
- **Null handling**: May or may not allow null (implementation-dependent)
- **Dynamic sizing**: Grows when load factor exceeded

## Time Complexity

| Operation    | Average | Worst  |
|--------------|---------|--------|
| Add          | O(1)    | O(n)   |
| Remove       | O(1)    | O(n)   |
| Contains     | O(1)    | O(n)   |
| Size         | O(1)    | O(1)   |
| Union        | O(m+n)  | O(m*n) |
| Intersection | O(min)  | O(m*n) |
| Difference   | O(m)    | O(m*n) |

*Worst case with poor hash function*

## Space Complexity

O(n) where n is the number of elements. Actual memory includes bucket array overhead and any collision handling structures.

## Operations

### Basic Operations

```pseudocode
function add(element) {
    index = hash(element) % this.capacity

    // Check if already exists
    for item in this.buckets[index] {
        if item == element {
            return false  // Already exists
        }
    }

    // Add new element
    this.buckets[index].append(element)
    this.size = this.size + 1

    if this.size > this.capacity * LOAD_FACTOR {
        resize()
    }

    return true
}

function remove(element) {
    index = hash(element) % this.capacity

    for i from 0 to this.buckets[index].length - 1 {
        if this.buckets[index][i] == element {
            this.buckets[index].removeAt(i)
            this.size = this.size - 1
            return true
        }
    }

    return false
}

function contains(element) {
    index = hash(element) % this.capacity

    for item in this.buckets[index] {
        if item == element {
            return true
        }
    }

    return false
}
```

### Set Operations

```pseudocode
function union(otherSet) {
    result = new HashSet()

    for element in this {
        result.add(element)
    }

    for element in otherSet {
        result.add(element)
    }

    return result
}

function intersection(otherSet) {
    result = new HashSet()

    // Iterate over smaller set for efficiency
    smaller = this.size <= otherSet.size ? this : otherSet
    larger = this.size <= otherSet.size ? otherSet : this

    for element in smaller {
        if larger.contains(element) {
            result.add(element)
        }
    }

    return result
}

function difference(otherSet) {
    result = new HashSet()

    for element in this {
        if not otherSet.contains(element) {
            result.add(element)
        }
    }

    return result
}

function symmetricDifference(otherSet) {
    result = new HashSet()

    for element in this {
        if not otherSet.contains(element) {
            result.add(element)
        }
    }

    for element in otherSet {
        if not this.contains(element) {
            result.add(element)
        }
    }

    return result
}

function isSubset(otherSet) {
    for element in this {
        if not otherSet.contains(element) {
            return false
        }
    }
    return true
}

function isSuperset(otherSet) {
    return otherSet.isSubset(this)
}

function isDisjoint(otherSet) {
    for element in this {
        if otherSet.contains(element) {
            return false
        }
    }
    return true
}
```

## Implementation

```pseudocode
class HashSet {
    buckets[]
    size = 0
    capacity
    LOAD_FACTOR = 0.75

    function constructor(initialCapacity = 16) {
        this.capacity = initialCapacity
        this.buckets = new Array(this.capacity)
        for i from 0 to this.capacity - 1 {
            this.buckets[i] = []
        }
    }

    function hash(element) {
        hashCode = element.hashCode()  // Or custom hash function
        return hashCode & 0x7FFFFFFF  // Ensure positive
    }

    function add(element) {
        index = hash(element) % this.capacity

        for item in this.buckets[index] {
            if item == element {
                return false
            }
        }

        this.buckets[index].append(element)
        this.size = this.size + 1

        if this.size > this.capacity * this.LOAD_FACTOR {
            resize(this.capacity * 2)
        }

        return true
    }

    function remove(element) {
        index = hash(element) % this.capacity

        for i from 0 to this.buckets[index].length - 1 {
            if this.buckets[index][i] == element {
                this.buckets[index].removeAt(i)
                this.size = this.size - 1
                return true
            }
        }

        return false
    }

    function contains(element) {
        index = hash(element) % this.capacity

        for item in this.buckets[index] {
            if item == element {
                return true
            }
        }

        return false
    }

    function resize(newCapacity) {
        oldBuckets = this.buckets
        this.capacity = newCapacity
        this.buckets = new Array(newCapacity)
        this.size = 0

        for i from 0 to newCapacity - 1 {
            this.buckets[i] = []
        }

        for bucket in oldBuckets {
            for element in bucket {
                add(element)
            }
        }
    }

    function toArray() {
        result = []
        for bucket in this.buckets {
            for element in bucket {
                result.append(element)
            }
        }
        return result
    }

    function clear() {
        for i from 0 to this.capacity - 1 {
            this.buckets[i] = []
        }
        this.size = 0
    }

    function isEmpty() {
        return this.size == 0
    }

    function size() {
        return this.size
    }

    // Iterator support
    function iterator() {
        elements = toArray()
        index = 0

        return {
            hasNext: function() {
                return index < elements.length
            },
            next: function() {
                return elements[index++]
            }
        }
    }

    // Set operations
    function union(other) {
        result = new HashSet()
        for element in this {
            result.add(element)
        }
        for element in other {
            result.add(element)
        }
        return result
    }

    function intersection(other) {
        result = new HashSet()
        smaller = this.size <= other.size ? this : other
        larger = this.size <= other.size ? other : this

        for element in smaller {
            if larger.contains(element) {
                result.add(element)
            }
        }
        return result
    }

    function difference(other) {
        result = new HashSet()
        for element in this {
            if not other.contains(element) {
                result.add(element)
            }
        }
        return result
    }

    function equals(other) {
        if this.size != other.size {
            return false
        }
        for element in this {
            if not other.contains(element) {
                return false
            }
        }
        return true
    }
}
```

## Classic Algorithms

### Find Duplicates

```pseudocode
function findDuplicates(array) {
    seen = new HashSet()
    duplicates = new HashSet()

    for element in array {
        if seen.contains(element) {
            duplicates.add(element)
        } else {
            seen.add(element)
        }
    }

    return duplicates.toArray()
}
```

### Two Sum Problem

```pseudocode
function twoSum(array, target) {
    seen = new HashSet()

    for num in array {
        complement = target - num
        if seen.contains(complement) {
            return [complement, num]
        }
        seen.add(num)
    }

    return null
}
```

### First Unique Element

```pseudocode
function firstUnique(array) {
    seen = new HashSet()
    duplicates = new HashSet()

    for element in array {
        if seen.contains(element) {
            duplicates.add(element)
        } else {
            seen.add(element)
        }
    }

    for element in array {
        if not duplicates.contains(element) {
            return element
        }
    }

    return null
}
```

### Longest Consecutive Sequence

```pseudocode
function longestConsecutive(nums) {
    numSet = new HashSet()
    for num in nums {
        numSet.add(num)
    }

    longest = 0

    for num in numSet {
        // Only start counting from sequence start
        if not numSet.contains(num - 1) {
            currentNum = num
            currentStreak = 1

            while numSet.contains(currentNum + 1) {
                currentNum = currentNum + 1
                currentStreak = currentStreak + 1
            }

            longest = max(longest, currentStreak)
        }
    }

    return longest
}
```

## Use Cases

- **Deduplication**: Remove duplicate elements from collections
- **Membership testing**: Fast "is element present?" queries
- **Set operations**: Union, intersection, difference
- **Visited tracking**: Graph traversal, avoiding cycles
- **Counting distinct**: Number of unique elements
- **Blacklisting/whitelisting**: Quick membership checks
- **Cache existence**: Check if item is cached

## Advantages

- **O(1) operations**: Fast add, remove, contains
- **Automatic deduplication**: Duplicates rejected
- **Set operations**: Natural support for union, intersection
- **Memory efficient**: No values stored (vs hash map)
- **Simple interface**: Add, remove, contains

## Disadvantages

- **Unordered**: No iteration order guarantee
- **No duplicates**: Cannot store multiple identical items
- **Worst case O(n)**: With poor hash function
- **No indexing**: Cannot access by position
- **Hash function required**: Elements must be hashable

## Comparison with Alternatives

| Aspect          | Hash Set    | Tree Set    | Sorted Array | Bit Set     |
|-----------------|-------------|-------------|--------------|-------------|
| Add             | O(1)        | O(log n)    | O(n)         | O(1)        |
| Contains        | O(1)        | O(log n)    | O(log n)     | O(1)        |
| Remove          | O(1)        | O(log n)    | O(n)         | O(1)        |
| Ordered         | No          | Yes         | Yes          | No          |
| Range query     | O(n)        | O(log n+k)  | O(log n+k)   | O(range)    |
| Memory          | Higher      | Higher      | Compact      | Very low*   |
| Element types   | Any hashable| Comparable  | Comparable   | Integers    |

*Bit set only for dense integer ranges

## Common Pitfalls

- **Mutable elements**: Modifying elements after adding breaks set
- **Equality vs identity**: Ensure equals() and hashCode() consistent
- **Null handling**: Some implementations don't allow null
- **Thread safety**: Standard hash set not thread-safe
- **Iteration during modification**: ConcurrentModificationException
- **Hash collisions**: Poor hash function degrades performance

## Related Structures

- **Hash Map**: Stores key-value pairs instead of just keys
- **Tree Set**: Sorted set using balanced BST
- **Linked Hash Set**: Maintains insertion order
- **Bit Set**: Space-efficient for integer ranges
- **Bloom Filter**: Probabilistic set membership
- **Concurrent Hash Set**: Thread-safe variant

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
