# Dynamic Array

## Overview

A dynamic array (also called resizable array, growable array, or vector) automatically grows when capacity is exceeded. It combines the O(1) access of fixed arrays with the flexibility to expand, making it the default choice for most collection needs.

## Properties

- **Automatic resizing**: Grows when full (typically doubles capacity)
- **Amortized O(1) append**: Most appends are O(1), occasional O(n) resize
- **Contiguous memory**: Maintains cache-friendly memory layout
- **Capacity vs size**: Capacity >= size; excess capacity reduces reallocations
- **Growth factor**: Usually 1.5x to 2x on resize

## Time Complexity

| Operation       | Average   | Worst    | Amortized |
|-----------------|-----------|----------|-----------|
| Access          | O(1)      | O(1)     | O(1)      |
| Search          | O(n)      | O(n)     | O(n)      |
| Append          | O(1)      | O(n)*    | O(1)      |
| Insert (middle) | O(n)      | O(n)     | O(n)      |
| Delete (end)    | O(1)      | O(1)     | O(1)      |
| Delete (middle) | O(n)      | O(n)     | O(n)      |

*O(n) when resize is triggered

## Space Complexity

O(n) with up to 2n memory usage due to capacity overhead. After resize, old array memory is freed, but temporarily both arrays exist during copy.

## Operations

### Append (Push Back)

Add element to end, resize if needed:

```pseudocode
function append(value) {
    if this.size >= this.capacity {
        resize(this.capacity * GROWTH_FACTOR)
    }
    this.data[this.size] = value
    this.size = this.size + 1
}
```

### Resize

Allocate new array and copy elements:

```pseudocode
function resize(newCapacity) {
    newData = allocate(newCapacity)

    for i from 0 to this.size - 1 {
        newData[i] = this.data[i]
    }

    free(this.data)
    this.data = newData
    this.capacity = newCapacity
}
```

### Insert at Position

```pseudocode
function insertAt(index, value) {
    if index < 0 or index > this.size {
        throw IndexOutOfBoundsError
    }

    if this.size >= this.capacity {
        resize(this.capacity * GROWTH_FACTOR)
    }

    // Shift elements right
    for i from this.size - 1 down to index {
        this.data[i + 1] = this.data[i]
    }

    this.data[index] = value
    this.size = this.size + 1
}
```

### Remove at Position

```pseudocode
function removeAt(index) {
    if index < 0 or index >= this.size {
        throw IndexOutOfBoundsError
    }

    value = this.data[index]

    // Shift elements left
    for i from index to this.size - 2 {
        this.data[i] = this.data[i + 1]
    }

    this.size = this.size - 1

    // Optional: shrink if too sparse
    if this.size < this.capacity / 4 {
        resize(this.capacity / 2)
    }

    return value
}
```

### Pop (Remove Last)

```pseudocode
function pop() {
    if this.size == 0 {
        throw EmptyArrayError
    }

    this.size = this.size - 1
    return this.data[this.size]
}
```

## Implementation

Complete dynamic array:

```pseudocode
class DynamicArray {
    data[]              // Underlying array
    size = 0            // Number of elements
    capacity = 0        // Current capacity
    GROWTH_FACTOR = 2   // How much to grow
    INITIAL_CAPACITY = 8

    function constructor(initialCapacity = INITIAL_CAPACITY) {
        this.data = allocate(initialCapacity)
        this.capacity = initialCapacity
        this.size = 0
    }

    function get(index) {
        if index < 0 or index >= this.size {
            throw IndexOutOfBoundsError
        }
        return this.data[index]
    }

    function set(index, value) {
        if index < 0 or index >= this.size {
            throw IndexOutOfBoundsError
        }
        this.data[index] = value
    }

    function append(value) {
        ensureCapacity(this.size + 1)
        this.data[this.size] = value
        this.size = this.size + 1
    }

    function prepend(value) {
        insertAt(0, value)
    }

    function insertAt(index, value) {
        if index < 0 or index > this.size {
            throw IndexOutOfBoundsError
        }

        ensureCapacity(this.size + 1)

        for i from this.size - 1 down to index {
            this.data[i + 1] = this.data[i]
        }

        this.data[index] = value
        this.size = this.size + 1
    }

    function removeAt(index) {
        if index < 0 or index >= this.size {
            throw IndexOutOfBoundsError
        }

        value = this.data[index]

        for i from index to this.size - 2 {
            this.data[i] = this.data[i + 1]
        }

        this.size = this.size - 1
        shrinkIfNeeded()
        return value
    }

    function pop() {
        if this.size == 0 {
            throw EmptyArrayError
        }
        this.size = this.size - 1
        value = this.data[this.size]
        shrinkIfNeeded()
        return value
    }

    function clear() {
        this.size = 0
        // Optionally reset capacity
    }

    function contains(value) {
        for i from 0 to this.size - 1 {
            if this.data[i] == value {
                return true
            }
        }
        return false
    }

    function indexOf(value) {
        for i from 0 to this.size - 1 {
            if this.data[i] == value {
                return i
            }
        }
        return -1
    }

    function isEmpty() {
        return this.size == 0
    }

    function length() {
        return this.size
    }

    // Private methods

    function ensureCapacity(minCapacity) {
        if minCapacity > this.capacity {
            newCapacity = max(this.capacity * GROWTH_FACTOR, minCapacity)
            resize(newCapacity)
        }
    }

    function shrinkIfNeeded() {
        if this.size > 0 and this.size < this.capacity / 4 {
            resize(max(this.capacity / 2, INITIAL_CAPACITY))
        }
    }

    function resize(newCapacity) {
        newData = allocate(newCapacity)

        for i from 0 to this.size - 1 {
            newData[i] = this.data[i]
        }

        free(this.data)
        this.data = newData
        this.capacity = newCapacity
    }
}
```

## Use Cases

- **General-purpose lists**: Default choice for ordered collections
- **Stacks**: Append/pop operations are O(1)
- **Buffers**: Growing data buffers for I/O
- **Building arrays**: When final size is unknown
- **Caching**: Variable-size result caching

## Advantages

- **Amortized O(1) append**: Efficient for growing collections
- **O(1) random access**: Same as fixed arrays
- **Cache efficient**: Contiguous memory layout
- **Memory efficient**: Adjusts to actual needs
- **Simple interface**: Familiar array semantics

## Disadvantages

- **Resize overhead**: Occasional O(n) copy operations
- **Memory spikes**: Temporarily uses 2x memory during resize
- **Wasted capacity**: May have unused allocated space
- **Expensive middle operations**: Insert/delete still O(n)

## Comparison with Alternatives

| Aspect               | Dynamic Array | Linked List | Deque         |
|----------------------|---------------|-------------|---------------|
| Access               | O(1)          | O(n)        | O(1)          |
| Append               | O(1) amort.   | O(1)        | O(1) amort.   |
| Prepend              | O(n)          | O(1)        | O(1) amort.   |
| Insert (middle)      | O(n)          | O(1)*       | O(n)          |
| Memory overhead      | Low           | High        | Medium        |
| Cache performance    | Excellent     | Poor        | Good          |

*After reaching the position

## Common Pitfalls

- **Forgetting amortization**: Single append can be O(n)
- **Growth factor too small**: 1.1x causes excessive reallocations
- **Growth factor too large**: 3x wastes memory
- **Not shrinking**: Arrays that grew large stay large
- **Iterator invalidation**: Resize invalidates pointers/references
- **Reserving capacity**: For known sizes, pre-allocate to avoid resizes

## Related Structures

- **Array**: Fixed-size foundation
- **Deque**: Efficient operations at both ends
- **Gap Buffer**: Optimized for localized insertions (text editors)
- **Rope**: For very large strings with frequent modifications
- **Circular Buffer**: Fixed-size with wrap-around behavior

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
