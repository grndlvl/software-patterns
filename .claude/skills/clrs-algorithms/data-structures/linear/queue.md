# Queue

## Overview

A queue is a First-In-First-Out (FIFO) data structure where elements are added at one end (rear/back) and removed from the other end (front). Like a line at a checkout counter, the first person to arrive is the first to be served. Queues are fundamental for scheduling, buffering, and breadth-first traversals.

## Properties

- **FIFO ordering**: First element added is first removed
- **Two access points**: Enqueue at rear, dequeue from front
- **Restricted operations**: Only enqueue, dequeue, and peek
- **Abstract data type**: Implementable via array or linked list
- **Natural ordering**: Preserves insertion order

## Time Complexity

| Operation | Average | Worst  |
|-----------|---------|--------|
| Enqueue   | O(1)    | O(1)*  |
| Dequeue   | O(1)    | O(1)   |
| Peek      | O(1)    | O(1)   |
| Search    | O(n)    | O(n)   |
| isEmpty   | O(1)    | O(1)   |

*O(n) for naive array implementation without circular buffer

## Space Complexity

O(n) where n is the number of elements. Circular array implementation uses fixed space efficiently.

## Operations

### Enqueue

Add element to rear:

```pseudocode
function enqueue(value) {
    // Circular array
    if this.size >= this.capacity {
        throw QueueFullError  // Or resize
    }
    this.data[this.rear] = value
    this.rear = (this.rear + 1) % this.capacity
    this.size = this.size + 1

    // Linked list
    // newNode = new Node(value)
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

### Dequeue

Remove and return front element:

```pseudocode
function dequeue() {
    if this.isEmpty() {
        throw QueueEmptyError
    }

    // Circular array
    value = this.data[this.front]
    this.data[this.front] = null
    this.front = (this.front + 1) % this.capacity
    this.size = this.size - 1
    return value

    // Linked list
    // value = this.head.data
    // this.head = this.head.next
    // if this.head == null {
    //     this.tail = null
    // }
    // this.size--
    // return value
}
```

### Peek (Front)

Return front element without removing:

```pseudocode
function peek() {
    if this.isEmpty() {
        throw QueueEmptyError
    }

    // Circular array
    return this.data[this.front]

    // Linked list
    // return this.head.data
}
```

## Implementation

### Circular Array Queue

```pseudocode
class CircularArrayQueue {
    data[]
    front = 0
    rear = 0
    size = 0
    capacity

    function constructor(capacity = 16) {
        this.data = allocate(capacity)
        this.capacity = capacity
    }

    function enqueue(value) {
        if this.size >= this.capacity {
            resize(this.capacity * 2)
        }

        this.data[this.rear] = value
        this.rear = (this.rear + 1) % this.capacity
        this.size = this.size + 1
    }

    function dequeue() {
        if this.size == 0 {
            throw QueueEmptyError
        }

        value = this.data[this.front]
        this.data[this.front] = null
        this.front = (this.front + 1) % this.capacity
        this.size = this.size - 1
        return value
    }

    function peek() {
        if this.size == 0 {
            throw QueueEmptyError
        }
        return this.data[this.front]
    }

    function isEmpty() {
        return this.size == 0
    }

    function isFull() {
        return this.size == this.capacity
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

### Linked List Queue

```pseudocode
class LinkedQueue {
    head = null
    tail = null
    size = 0

    function enqueue(value) {
        newNode = new Node(value)

        if this.tail != null {
            this.tail.next = newNode
        }
        this.tail = newNode

        if this.head == null {
            this.head = newNode
        }

        this.size = this.size + 1
    }

    function dequeue() {
        if this.head == null {
            throw QueueEmptyError
        }

        value = this.head.data
        this.head = this.head.next

        if this.head == null {
            this.tail = null
        }

        this.size = this.size - 1
        return value
    }

    function peek() {
        if this.head == null {
            throw QueueEmptyError
        }
        return this.head.data
    }

    function isEmpty() {
        return this.head == null
    }

    function size() {
        return this.size
    }
}
```

### Queue with Two Stacks

```pseudocode
class TwoStackQueue {
    inbox = new Stack()
    outbox = new Stack()

    function enqueue(value) {
        inbox.push(value)
    }

    function dequeue() {
        if outbox.isEmpty() {
            while !inbox.isEmpty() {
                outbox.push(inbox.pop())
            }
        }

        if outbox.isEmpty() {
            throw QueueEmptyError
        }

        return outbox.pop()
    }

    function peek() {
        if outbox.isEmpty() {
            while !inbox.isEmpty() {
                outbox.push(inbox.pop())
            }
        }

        if outbox.isEmpty() {
            throw QueueEmptyError
        }

        return outbox.peek()
    }

    function isEmpty() {
        return inbox.isEmpty() and outbox.isEmpty()
    }
}
```

## Variants

### Priority Queue

Elements dequeued by priority, not arrival order:

```pseudocode
class PriorityQueue {
    // Usually implemented with heap (see heap.md)

    function enqueue(value, priority) {
        // Add to heap with priority
    }

    function dequeue() {
        // Remove highest priority element
    }
}
```

### Blocking Queue

Thread-safe queue that blocks on empty/full:

```pseudocode
class BlockingQueue {
    queue = new Queue()
    lock = new Mutex()
    notEmpty = new Condition()
    notFull = new Condition()
    capacity

    function enqueue(value) {
        lock.acquire()
        while queue.size() >= capacity {
            notFull.wait(lock)
        }
        queue.enqueue(value)
        notEmpty.signal()
        lock.release()
    }

    function dequeue() {
        lock.acquire()
        while queue.isEmpty() {
            notEmpty.wait(lock)
        }
        value = queue.dequeue()
        notFull.signal()
        lock.release()
        return value
    }
}
```

## Classic Algorithms

### BFS (Breadth-First Search)

```pseudocode
function bfs(graph, start) {
    visited = new Set()
    queue = new Queue()

    queue.enqueue(start)
    visited.add(start)

    while !queue.isEmpty() {
        node = queue.dequeue()
        process(node)

        for neighbor in graph.neighbors(node) {
            if neighbor not in visited {
                visited.add(neighbor)
                queue.enqueue(neighbor)
            }
        }
    }
}
```

### Level-Order Tree Traversal

```pseudocode
function levelOrder(root) {
    if root == null {
        return
    }

    queue = new Queue()
    queue.enqueue(root)

    while !queue.isEmpty() {
        levelSize = queue.size()

        for i from 0 to levelSize - 1 {
            node = queue.dequeue()
            process(node)

            if node.left != null {
                queue.enqueue(node.left)
            }
            if node.right != null {
                queue.enqueue(node.right)
            }
        }
    }
}
```

## Use Cases

- **Task scheduling**: OS process queues, print spoolers
- **BFS algorithms**: Graph traversal, shortest path in unweighted graphs
- **Buffering**: I/O buffers, streaming data
- **Message queues**: Inter-process communication
- **Request handling**: Web server request queues
- **Simulation**: Modeling real-world queues (banks, traffic)
- **Cache**: LRU cache eviction order

## Advantages

- **Fair ordering**: FIFO ensures first-come, first-served
- **Constant time operations**: O(1) for all primary operations
- **Simple interface**: Easy to understand and use
- **Natural modeling**: Maps to real-world queuing scenarios
- **Thread-safe versions**: Well-suited for producer-consumer patterns

## Disadvantages

- **No random access**: Can only access front element
- **Fixed capacity** (array-based): May waste space or need resize
- **No priority handling**: Basic queue ignores importance
- **Memory overhead** (linked list): Extra pointers per element

## Comparison with Alternatives

| Aspect          | Queue        | Stack        | Deque         | Priority Queue |
|-----------------|--------------|--------------|---------------|----------------|
| Access pattern  | FIFO         | LIFO         | Both ends     | By priority    |
| Enqueue/Dequeue | Opposite ends| Same end     | Either end    | Any/Top        |
| Use case        | Scheduling   | Backtracking | Flexible      | Prioritization |
| Implementation  | Simple       | Simple       | Moderate      | Heap-based     |

## Common Pitfalls

- **Queue overflow**: Bounded queues can fill up
- **Queue underflow**: Dequeuing from empty queue
- **Circular array confusion**: Off-by-one errors in wrap-around logic
- **Memory waste**: Linear array dequeue leaves unused space
- **Forgetting tail update**: Linked list tail becomes stale
- **Blocking deadlock**: Two threads waiting on each other

## Related Structures

- **Stack**: LIFO instead of FIFO
- **Deque**: Double-ended queue, flexible
- **Priority Queue**: Dequeue by priority (usually heap-based)
- **Circular Buffer**: Fixed-size queue with wrap-around
- **Blocking Queue**: Thread-safe with wait semantics

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
