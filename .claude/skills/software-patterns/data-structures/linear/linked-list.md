# Linked List

## Overview

A linked list is a linear data structure where elements (nodes) are connected via pointers. Unlike arrays, elements are not stored contiguously; each node contains data and reference(s) to other nodes. This enables O(1) insertions and deletions at known positions but sacrifices random access.

## Variants

- **Singly Linked List**: Each node points to next node only
- **Doubly Linked List**: Each node points to both next and previous
- **Circular Linked List**: Last node points back to first (can be singly or doubly)

## Properties

- **Dynamic size**: Grows and shrinks as needed
- **Non-contiguous**: Nodes scattered in memory
- **Sequential access**: Must traverse from head to reach element
- **Pointer overhead**: Each node stores 1-2 pointers
- **No capacity limits**: Size limited only by available memory

## Time Complexity

| Operation        | Singly  | Doubly  | Notes                    |
|------------------|---------|---------|--------------------------|
| Access           | O(n)    | O(n)    | Must traverse            |
| Search           | O(n)    | O(n)    | Linear scan              |
| Insert (head)    | O(1)    | O(1)    | Just update pointers     |
| Insert (tail)    | O(n)/O(1)* | O(1) | *O(1) with tail pointer  |
| Insert (middle)  | O(n)    | O(n)    | O(1) if position known   |
| Delete (head)    | O(1)    | O(1)    | Update head pointer      |
| Delete (tail)    | O(n)    | O(1)    | Singly needs prev node   |
| Delete (middle)  | O(n)    | O(n)    | O(1) if node reference   |

## Space Complexity

O(n) for n elements. Additional pointer overhead:
- Singly: 1 pointer per node
- Doubly: 2 pointers per node

## Operations

### Node Structure

```pseudocode
// Singly Linked
class SinglyNode {
    data
    next = null
}

// Doubly Linked
class DoublyNode {
    data
    prev = null
    next = null
}
```

### Insert at Head

```pseudocode
function insertAtHead(value) {
    newNode = new Node(value)
    newNode.next = this.head

    if this.head != null and isDoubly {
        this.head.prev = newNode
    }

    this.head = newNode

    if this.tail == null {
        this.tail = newNode
    }

    this.size = this.size + 1
}
```

### Insert at Tail

```pseudocode
function insertAtTail(value) {
    newNode = new Node(value)

    if this.tail == null {
        this.head = newNode
        this.tail = newNode
    } else {
        this.tail.next = newNode
        if isDoubly {
            newNode.prev = this.tail
        }
        this.tail = newNode
    }

    this.size = this.size + 1
}
```

### Insert After Node

```pseudocode
function insertAfter(node, value) {
    if node == null {
        throw NullPointerError
    }

    newNode = new Node(value)
    newNode.next = node.next

    if isDoubly {
        newNode.prev = node
        if node.next != null {
            node.next.prev = newNode
        }
    }

    node.next = newNode

    if node == this.tail {
        this.tail = newNode
    }

    this.size = this.size + 1
}
```

### Delete Head

```pseudocode
function deleteHead() {
    if this.head == null {
        throw EmptyListError
    }

    value = this.head.data
    this.head = this.head.next

    if this.head != null and isDoubly {
        this.head.prev = null
    }

    if this.head == null {
        this.tail = null
    }

    this.size = this.size - 1
    return value
}
```

### Delete Tail (Doubly)

```pseudocode
function deleteTail() {
    if this.tail == null {
        throw EmptyListError
    }

    value = this.tail.data

    if this.tail.prev != null {
        this.tail.prev.next = null
        this.tail = this.tail.prev
    } else {
        this.head = null
        this.tail = null
    }

    this.size = this.size - 1
    return value
}
```

### Delete Node

```pseudocode
function deleteNode(node) {
    if node == null {
        return
    }

    // Doubly linked - O(1)
    if isDoubly {
        if node.prev != null {
            node.prev.next = node.next
        } else {
            this.head = node.next
        }

        if node.next != null {
            node.next.prev = node.prev
        } else {
            this.tail = node.prev
        }
    }
    // Singly linked - need to find previous node - O(n)
    else {
        if node == this.head {
            this.head = node.next
        } else {
            prev = this.head
            while prev.next != node {
                prev = prev.next
            }
            prev.next = node.next
        }

        if node == this.tail {
            this.tail = prev  // undefined for singly without traversal
        }
    }

    this.size = this.size - 1
}
```

### Search

```pseudocode
function find(value) {
    current = this.head

    while current != null {
        if current.data == value {
            return current
        }
        current = current.next
    }

    return null
}
```

### Reverse

```pseudocode
function reverse() {
    prev = null
    current = this.head
    this.tail = this.head

    while current != null {
        nextNode = current.next
        current.next = prev

        if isDoubly {
            current.prev = nextNode
        }

        prev = current
        current = nextNode
    }

    this.head = prev
}
```

## Implementation

### Singly Linked List

```pseudocode
class SinglyLinkedList {
    head = null
    tail = null
    size = 0

    function insertAtHead(value) {
        newNode = new SinglyNode(value)
        newNode.next = this.head
        this.head = newNode

        if this.tail == null {
            this.tail = newNode
        }
        this.size++
    }

    function insertAtTail(value) {
        newNode = new SinglyNode(value)

        if this.tail == null {
            this.head = newNode
            this.tail = newNode
        } else {
            this.tail.next = newNode
            this.tail = newNode
        }
        this.size++
    }

    function deleteAtHead() {
        if this.head == null {
            return null
        }

        value = this.head.data
        this.head = this.head.next

        if this.head == null {
            this.tail = null
        }

        this.size--
        return value
    }

    function get(index) {
        if index < 0 or index >= this.size {
            throw IndexOutOfBoundsError
        }

        current = this.head
        for i from 0 to index - 1 {
            current = current.next
        }
        return current.data
    }

    function isEmpty() {
        return this.size == 0
    }
}
```

### Doubly Linked List

```pseudocode
class DoublyLinkedList {
    head = null
    tail = null
    size = 0

    function insertAtHead(value) {
        newNode = new DoublyNode(value)

        if this.head == null {
            this.head = newNode
            this.tail = newNode
        } else {
            newNode.next = this.head
            this.head.prev = newNode
            this.head = newNode
        }
        this.size++
    }

    function insertAtTail(value) {
        newNode = new DoublyNode(value)

        if this.tail == null {
            this.head = newNode
            this.tail = newNode
        } else {
            newNode.prev = this.tail
            this.tail.next = newNode
            this.tail = newNode
        }
        this.size++
    }

    function deleteAtHead() {
        if this.head == null {
            return null
        }

        value = this.head.data
        this.head = this.head.next

        if this.head != null {
            this.head.prev = null
        } else {
            this.tail = null
        }

        this.size--
        return value
    }

    function deleteAtTail() {
        if this.tail == null {
            return null
        }

        value = this.tail.data
        this.tail = this.tail.prev

        if this.tail != null {
            this.tail.next = null
        } else {
            this.head = null
        }

        this.size--
        return value
    }

    function deleteNode(node) {
        if node.prev != null {
            node.prev.next = node.next
        } else {
            this.head = node.next
        }

        if node.next != null {
            node.next.prev = node.prev
        } else {
            this.tail = node.prev
        }

        this.size--
    }
}
```

### Circular Linked List

```pseudocode
class CircularLinkedList {
    head = null
    size = 0

    function insert(value) {
        newNode = new SinglyNode(value)

        if this.head == null {
            this.head = newNode
            newNode.next = newNode  // Points to itself
        } else {
            // Find last node
            current = this.head
            while current.next != this.head {
                current = current.next
            }
            current.next = newNode
            newNode.next = this.head
        }
        this.size++
    }

    function traverse(callback) {
        if this.head == null {
            return
        }

        current = this.head
        do {
            callback(current.data)
            current = current.next
        } while current != this.head
    }
}
```

## Use Cases

- **Undo functionality**: Each action points to previous state
- **Music playlists**: Circular list for repeat mode
- **Browser history**: Back/forward navigation (doubly linked)
- **Memory allocation**: Free block lists in memory managers
- **Polynomial representation**: Terms as nodes
- **LRU Cache**: Combined with hash map for O(1) operations
- **Task scheduling**: Round-robin with circular list

## Advantages

- **Dynamic size**: No pre-allocation needed
- **O(1) insertions/deletions**: At known positions
- **No wasted space**: Exactly as much memory as needed
- **No shifting**: Unlike arrays, no element movement
- **Easy concatenation**: Just link tail to head

## Disadvantages

- **No random access**: Must traverse sequentially
- **Memory overhead**: Extra pointer(s) per element
- **Poor cache locality**: Nodes scattered in memory
- **Extra complexity**: Pointer management is error-prone
- **Reverse traversal**: Expensive for singly linked (need doubly)

## Comparison with Alternatives

| Aspect             | Singly Linked | Doubly Linked | Array      | Dynamic Array |
|--------------------|---------------|---------------|------------|---------------|
| Access             | O(n)          | O(n)          | O(1)       | O(1)          |
| Insert (head)      | O(1)          | O(1)          | O(n)       | O(n)          |
| Insert (tail)      | O(1)*         | O(1)          | O(1)       | O(1) amort.   |
| Insert (middle)    | O(n)          | O(n)          | O(n)       | O(n)          |
| Delete (head)      | O(1)          | O(1)          | O(n)       | O(n)          |
| Delete (tail)      | O(n)          | O(1)          | O(1)       | O(1)          |
| Memory per element | +1 pointer    | +2 pointers   | None       | None          |
| Cache efficiency   | Poor          | Poor          | Excellent  | Excellent     |

*With tail pointer

## Common Pitfalls

- **Losing references**: Reassigning head/tail without saving data
- **Memory leaks**: Not freeing deleted nodes (in manual memory languages)
- **Null pointer errors**: Not checking for null before accessing next/prev
- **Off-by-one**: Traversing one node too many or too few
- **Circular references**: Accidentally creating cycles in non-circular lists
- **Forgetting tail updates**: Tail pointer becomes stale after operations

## Related Structures

- **Skip List**: Linked list with express lanes for O(log n) search
- **XOR Linked List**: Memory-efficient doubly linked using XOR of addresses
- **Unrolled Linked List**: Each node contains small array of elements
- **Self-organizing List**: Moves frequently accessed elements to front

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
