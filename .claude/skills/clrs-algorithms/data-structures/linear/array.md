# Array

## Overview

An array is a contiguous block of memory that stores a fixed number of elements of the same type. It provides constant-time access to any element via its index, making it the most fundamental data structure in computer science.

## Properties

- **Fixed size**: Size determined at creation time
- **Homogeneous**: All elements are the same type
- **Contiguous memory**: Elements stored in adjacent memory locations
- **Zero-indexed**: First element at index 0 (in most languages)
- **Random access**: Any element accessible in O(1) time

## Time Complexity

| Operation      | Average | Worst  |
|----------------|---------|--------|
| Access         | O(1)    | O(1)   |
| Search         | O(n)    | O(n)   |
| Insert (end)   | O(1)*   | O(1)*  |
| Insert (middle)| O(n)    | O(n)   |
| Delete (end)   | O(1)    | O(1)   |
| Delete (middle)| O(n)    | O(n)   |

*Only if space is pre-allocated

## Space Complexity

O(n) where n is the number of elements. Memory overhead is minimal since arrays store only the data itself with no per-element metadata.

## Operations

### Access by Index

Retrieve element at a specific position:

```pseudocode
function get(array, index) {
    if index < 0 or index >= array.length {
        throw IndexOutOfBoundsError
    }
    return array[index]
}
```

### Linear Search

Find element in unsorted array:

```pseudocode
function linearSearch(array, target) {
    for i from 0 to array.length - 1 {
        if array[i] == target {
            return i
        }
    }
    return -1  // Not found
}
```

### Binary Search (Sorted Array)

Find element in sorted array:

```pseudocode
function binarySearch(array, target) {
    left = 0
    right = array.length - 1

    while left <= right {
        mid = left + (right - left) / 2  // Avoid overflow

        if array[mid] == target {
            return mid
        } else if array[mid] < target {
            left = mid + 1
        } else {
            right = mid - 1
        }
    }
    return -1  // Not found
}
```

### Insert at Position

```pseudocode
function insertAt(array, index, value) {
    if index < 0 or index > array.length {
        throw IndexOutOfBoundsError
    }

    // Shift elements right
    for i from array.length - 1 down to index {
        array[i + 1] = array[i]
    }

    array[index] = value
    array.length = array.length + 1
}
```

### Delete at Position

```pseudocode
function deleteAt(array, index) {
    if index < 0 or index >= array.length {
        throw IndexOutOfBoundsError
    }

    value = array[index]

    // Shift elements left
    for i from index to array.length - 2 {
        array[i] = array[i + 1]
    }

    array.length = array.length - 1
    return value
}
```

## Implementation

Basic array operations wrapper:

```pseudocode
class Array {
    data[]      // Raw memory block
    capacity    // Maximum size
    length      // Current number of elements

    function constructor(capacity) {
        this.data = allocate(capacity)
        this.capacity = capacity
        this.length = 0
    }

    function get(index) {
        if index < 0 or index >= this.length {
            throw IndexOutOfBoundsError
        }
        return this.data[index]
    }

    function set(index, value) {
        if index < 0 or index >= this.length {
            throw IndexOutOfBoundsError
        }
        this.data[index] = value
    }

    function append(value) {
        if this.length >= this.capacity {
            throw ArrayFullError
        }
        this.data[this.length] = value
        this.length = this.length + 1
    }

    function size() {
        return this.length
    }
}
```

## Use Cases

- **Storing collections**: When size is known and fixed
- **Lookup tables**: Fast O(1) access by index
- **Buffer storage**: Audio, video, network data
- **Matrix operations**: 2D arrays for image processing, games
- **Implementing other structures**: Stacks, queues, heaps built on arrays

## Advantages

- **Constant-time access**: O(1) read/write by index
- **Memory efficient**: No overhead per element
- **Cache friendly**: Contiguous memory improves CPU cache performance
- **Simple**: Easy to understand and implement
- **Predictable**: Fixed memory footprint

## Disadvantages

- **Fixed size**: Cannot grow without creating new array
- **Expensive insertions/deletions**: Middle operations are O(n)
- **Wasted space**: May allocate more than needed
- **Single type**: All elements must be same type (in typed languages)

## Comparison with Alternatives

| Aspect            | Array           | Linked List     | Dynamic Array   |
|-------------------|-----------------|-----------------|-----------------|
| Access            | O(1)            | O(n)            | O(1)            |
| Insert (middle)   | O(n)            | O(1)*           | O(n)            |
| Memory overhead   | None            | Pointer/element | Amortized       |
| Cache performance | Excellent       | Poor            | Excellent       |
| Size flexibility  | Fixed           | Dynamic         | Dynamic         |

*After finding the position

## Common Pitfalls

- **Off-by-one errors**: Remember arrays are 0-indexed; last element is at `length - 1`
- **Buffer overflow**: Writing beyond array bounds causes memory corruption
- **Uninitialized access**: Reading from uninitialized indices yields garbage
- **Forgetting bounds checks**: Always validate indices before access
- **Integer overflow**: When calculating indices, especially `(left + right) / 2`

## Related Structures

- **Dynamic Array**: Resizable version that grows automatically
- **Multi-dimensional Array**: Arrays of arrays for matrices/tensors
- **Circular Buffer**: Fixed-size array with wrap-around indices
- **String**: Often implemented as character arrays

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
