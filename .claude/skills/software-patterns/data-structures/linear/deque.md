# Deque (Double-Ended Queue)

## Overview

A deque (pronounced "deck") is a double-ended queue that allows insertion and deletion at both the front and rear. It combines the capabilities of both stacks and queues, making it a versatile data structure for scenarios requiring flexible access patterns.

## Properties

- **Double-ended access**: Operations at both front and rear
- **Combines stack and queue**: Can function as either
- **Ordered collection**: Maintains insertion order
- **Abstract data type**: Multiple implementation strategies
- **Symmetrical operations**: push/pop at both ends

## Time Complexity

| Operation      | Average | Worst  |
|----------------|---------|--------|
| Push Front     | O(1)    | O(1)*  |
| Push Back      | O(1)    | O(1)*  |
| Pop Front      | O(1)    | O(1)   |
| Pop Back       | O(1)    | O(1)   |
| Peek Front     | O(1)    | O(1)   |
| Peek Back      | O(1)    | O(1)   |
| Access (index) | O(1)    | O(1)   |
| Search         | O(n)    | O(n)   |

*Amortized O(1) for dynamic implementations

## Space Complexity

O(n) where n is the number of elements. Implementation-dependent:
- Circular array: Fixed capacity with possible unused slots
- Doubly linked list: Two pointers per element overhead
- Block-based: Small fixed overhead per block

## Operations

### Push Front

Add element to front:

```pseudocode
function pushFront(value) {
    // Circular array
    if this.size >= this.capacity {
        resize(this.capacity * 2)
    }
    this.front = (this.front - 1 + this.capacity) % this.capacity
    this.data[this.front] = value
    this.size = this.size + 1

    // Doubly linked list
    // newNode = new Node(value)
    // newNode.next = this.head
    // if this.head != null {
    //     this.head.prev = newNode
    // }
    // this.head = newNode
    // if this.tail == null {
    //     this.tail = newNode
    // }
    // this.size++
}
```

### Push Back

Add element to rear:

```pseudocode
function pushBack(value) {
    // Circular array
    if this.size >= this.capacity {
        resize(this.capacity * 2)
    }
    this.data[this.rear] = value
    this.rear = (this.rear + 1) % this.capacity
    this.size = this.size + 1

    // Doubly linked list
    // newNode = new Node(value)
    // newNode.prev = this.tail
    // if this.tail != null {
    //     this.tail.next = newNode
    // }
    // this.tail = newNode
    // if this.head == null {
    //     this.head = newNode
    // }
    // this.size++
}
```

### Pop Front

Remove and return front element:

```pseudocode
function popFront() {
    if this.isEmpty() {
        throw DequeEmptyError
    }

    // Circular array
    value = this.data[this.front]
    this.data[this.front] = null
    this.front = (this.front + 1) % this.capacity
    this.size = this.size - 1
    return value

    // Doubly linked list
    // value = this.head.data
    // this.head = this.head.next
    // if this.head != null {
    //     this.head.prev = null
    // } else {
    //     this.tail = null
    // }
    // this.size--
    // return value
}
```

### Pop Back

Remove and return rear element:

```pseudocode
function popBack() {
    if this.isEmpty() {
        throw DequeEmptyError
    }

    // Circular array
    this.rear = (this.rear - 1 + this.capacity) % this.capacity
    value = this.data[this.rear]
    this.data[this.rear] = null
    this.size = this.size - 1
    return value

    // Doubly linked list
    // value = this.tail.data
    // this.tail = this.tail.prev
    // if this.tail != null {
    //     this.tail.next = null
    // } else {
    //     this.head = null
    // }
    // this.size--
    // return value
}
```

## Implementation

### Circular Array Deque

```pseudocode
class CircularArrayDeque {
    data[]
    front = 0
    rear = 0
    size = 0
    capacity

    function constructor(capacity = 16) {
        this.data = allocate(capacity)
        this.capacity = capacity
    }

    function pushFront(value) {
        if this.size >= this.capacity {
            resize(this.capacity * 2)
        }

        this.front = (this.front - 1 + this.capacity) % this.capacity
        this.data[this.front] = value
        this.size = this.size + 1
    }

    function pushBack(value) {
        if this.size >= this.capacity {
            resize(this.capacity * 2)
        }

        this.data[this.rear] = value
        this.rear = (this.rear + 1) % this.capacity
        this.size = this.size + 1
    }

    function popFront() {
        if this.size == 0 {
            throw DequeEmptyError
        }

        value = this.data[this.front]
        this.data[this.front] = null
        this.front = (this.front + 1) % this.capacity
        this.size = this.size - 1
        return value
    }

    function popBack() {
        if this.size == 0 {
            throw DequeEmptyError
        }

        this.rear = (this.rear - 1 + this.capacity) % this.capacity
        value = this.data[this.rear]
        this.data[this.rear] = null
        this.size = this.size - 1
        return value
    }

    function peekFront() {
        if this.size == 0 {
            throw DequeEmptyError
        }
        return this.data[this.front]
    }

    function peekBack() {
        if this.size == 0 {
            throw DequeEmptyError
        }
        return this.data[(this.rear - 1 + this.capacity) % this.capacity]
    }

    function get(index) {
        if index < 0 or index >= this.size {
            throw IndexOutOfBoundsError
        }
        return this.data[(this.front + index) % this.capacity]
    }

    function isEmpty() {
        return this.size == 0
    }

    function size() {
        return this.size
    }

    function resize(newCapacity) {
        newData = allocate(newCapacity)

        for i from 0 to this.size - 1 {
            newData[i] = this.data[(this.front + i) % this.capacity]
        }

        this.data = newData
        this.front = 0
        this.rear = this.size
        this.capacity = newCapacity
    }
}
```

### Doubly Linked List Deque

```pseudocode
class LinkedDeque {
    head = null
    tail = null
    size = 0

    function pushFront(value) {
        newNode = new DoublyNode(value)

        if this.head == null {
            this.head = newNode
            this.tail = newNode
        } else {
            newNode.next = this.head
            this.head.prev = newNode
            this.head = newNode
        }

        this.size = this.size + 1
    }

    function pushBack(value) {
        newNode = new DoublyNode(value)

        if this.tail == null {
            this.head = newNode
            this.tail = newNode
        } else {
            newNode.prev = this.tail
            this.tail.next = newNode
            this.tail = newNode
        }

        this.size = this.size + 1
    }

    function popFront() {
        if this.head == null {
            throw DequeEmptyError
        }

        value = this.head.data
        this.head = this.head.next

        if this.head != null {
            this.head.prev = null
        } else {
            this.tail = null
        }

        this.size = this.size - 1
        return value
    }

    function popBack() {
        if this.tail == null {
            throw DequeEmptyError
        }

        value = this.tail.data
        this.tail = this.tail.prev

        if this.tail != null {
            this.tail.next = null
        } else {
            this.head = null
        }

        this.size = this.size - 1
        return value
    }

    function peekFront() {
        if this.head == null {
            throw DequeEmptyError
        }
        return this.head.data
    }

    function peekBack() {
        if this.tail == null {
            throw DequeEmptyError
        }
        return this.tail.data
    }

    function isEmpty() {
        return this.size == 0
    }

    function size() {
        return this.size
    }
}
```

## Variants

### Input-Restricted Deque

Only allows insertion at one end:

```pseudocode
class InputRestrictedDeque {
    // Can only push at rear
    function pushBack(value) { ... }

    // Can pop from both ends
    function popFront() { ... }
    function popBack() { ... }
}
```

### Output-Restricted Deque

Only allows removal from one end:

```pseudocode
class OutputRestrictedDeque {
    // Can push at both ends
    function pushFront(value) { ... }
    function pushBack(value) { ... }

    // Can only pop from front
    function popFront() { ... }
}
```

## Classic Algorithms

### Sliding Window Maximum

```pseudocode
function slidingWindowMax(array, k) {
    result = []
    deque = new Deque()  // Stores indices

    for i from 0 to array.length - 1 {
        // Remove indices outside window
        while !deque.isEmpty() and deque.peekFront() <= i - k {
            deque.popFront()
        }

        // Remove smaller elements (they'll never be max)
        while !deque.isEmpty() and array[deque.peekBack()] < array[i] {
            deque.popBack()
        }

        deque.pushBack(i)

        // Window is complete
        if i >= k - 1 {
            result.append(array[deque.peekFront()])
        }
    }

    return result
}
```

### Palindrome Check

```pseudocode
function isPalindrome(str) {
    deque = new Deque()

    for char in str {
        if isAlphanumeric(char) {
            deque.pushBack(toLowerCase(char))
        }
    }

    while deque.size() > 1 {
        if deque.popFront() != deque.popBack() {
            return false
        }
    }

    return true
}
```

## Use Cases

- **Sliding window algorithms**: Maximum/minimum in window
- **Palindrome checking**: Compare from both ends
- **Undo/redo with limit**: Fixed-size history
- **Work stealing**: Thread pools steal from both ends
- **A-Steal scheduler**: Parallel task scheduling
- **Browser history**: Navigate forward and back
- **Text editors**: Cursor movement optimization

## Advantages

- **Flexible access**: Operations at both ends
- **Constant time operations**: O(1) for all primary operations
- **Can act as stack or queue**: Versatile usage
- **Random access** (array-based): O(1) indexing
- **Efficient sliding window**: Natural fit for window problems

## Disadvantages

- **More complex**: Than simple stack or queue
- **Memory overhead** (linked list): Two pointers per element
- **Cache behavior** (linked list): Non-contiguous memory
- **Middle operations**: Still O(n) for insertion/deletion

## Comparison with Alternatives

| Aspect          | Deque         | Stack        | Queue        |
|-----------------|---------------|--------------|--------------|
| Push front      | O(1)          | O(n)*        | O(n)*        |
| Push back       | O(1)          | O(1)         | O(1)         |
| Pop front       | O(1)          | O(n)*        | O(1)         |
| Pop back        | O(1)          | O(1)         | O(n)*        |
| Flexibility     | High          | Low          | Low          |
| Implementation  | Complex       | Simple       | Simple       |

*Would require additional structure or rebuilding

## Common Pitfalls

- **Modular arithmetic errors**: Wrap-around logic is tricky
- **Empty deque access**: Always check before peek/pop
- **Resize logic**: Must preserve element order
- **Index calculation**: Converting logical to physical indices
- **Forgetting both ends**: Updates must maintain both head and tail

## Related Structures

- **Stack**: Deque restricted to one end
- **Queue**: Deque with insertion at one end, removal at other
- **Circular Buffer**: Fixed-size deque
- **Min-Max Deque**: Tracks min and max efficiently
- **Steque**: Stack-ended queue (push both, pop front only)

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
