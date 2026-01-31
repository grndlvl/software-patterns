# Stack

## Overview

A stack is a Last-In-First-Out (LIFO) data structure where elements are added and removed from the same end (the "top"). Think of a stack of plates: you can only add or remove from the top. This simple constraint makes stacks ideal for tracking state, parsing, and backtracking algorithms.

## Properties

- **LIFO ordering**: Last element added is first removed
- **Single access point**: All operations happen at the top
- **Restricted operations**: Only push, pop, and peek
- **Abstract data type**: Can be implemented with array or linked list
- **Recursive nature**: Naturally models recursive call patterns

## Time Complexity

| Operation | Average | Worst  |
|-----------|---------|--------|
| Push      | O(1)    | O(1)*  |
| Pop       | O(1)    | O(1)   |
| Peek      | O(1)    | O(1)   |
| Search    | O(n)    | O(n)   |
| isEmpty   | O(1)    | O(1)   |

*O(n) if using dynamic array that needs resize

## Space Complexity

O(n) where n is the number of elements. Implementation-dependent overhead:
- Array-based: Minimal overhead, possible unused capacity
- Linked list-based: One pointer per element

## Operations

### Push

Add element to top:

```pseudocode
function push(value) {
    // Array-based
    if this.size >= this.capacity {
        resize()  // Or throw StackOverflowError
    }
    this.data[this.size] = value
    this.size = this.size + 1

    // Linked list-based
    // newNode = new Node(value)
    // newNode.next = this.top
    // this.top = newNode
    // this.size++
}
```

### Pop

Remove and return top element:

```pseudocode
function pop() {
    if this.isEmpty() {
        throw StackUnderflowError
    }

    // Array-based
    this.size = this.size - 1
    return this.data[this.size]

    // Linked list-based
    // value = this.top.data
    // this.top = this.top.next
    // this.size--
    // return value
}
```

### Peek (Top)

Return top element without removing:

```pseudocode
function peek() {
    if this.isEmpty() {
        throw StackEmptyError
    }

    // Array-based
    return this.data[this.size - 1]

    // Linked list-based
    // return this.top.data
}
```

### isEmpty

```pseudocode
function isEmpty() {
    return this.size == 0
}
```

## Implementation

### Array-Based Stack

```pseudocode
class ArrayStack {
    data[]
    size = 0
    capacity

    function constructor(capacity = 16) {
        this.data = allocate(capacity)
        this.capacity = capacity
    }

    function push(value) {
        if this.size >= this.capacity {
            // Option 1: Resize
            newCapacity = this.capacity * 2
            newData = allocate(newCapacity)
            for i from 0 to this.size - 1 {
                newData[i] = this.data[i]
            }
            this.data = newData
            this.capacity = newCapacity

            // Option 2: Throw error
            // throw StackOverflowError
        }

        this.data[this.size] = value
        this.size = this.size + 1
    }

    function pop() {
        if this.size == 0 {
            throw StackUnderflowError
        }

        this.size = this.size - 1
        value = this.data[this.size]
        this.data[this.size] = null  // Help garbage collection
        return value
    }

    function peek() {
        if this.size == 0 {
            throw StackEmptyError
        }
        return this.data[this.size - 1]
    }

    function isEmpty() {
        return this.size == 0
    }

    function size() {
        return this.size
    }

    function clear() {
        this.size = 0
    }
}
```

### Linked List-Based Stack

```pseudocode
class LinkedStack {
    top = null
    size = 0

    function push(value) {
        newNode = new Node(value)
        newNode.next = this.top
        this.top = newNode
        this.size = this.size + 1
    }

    function pop() {
        if this.top == null {
            throw StackUnderflowError
        }

        value = this.top.data
        this.top = this.top.next
        this.size = this.size - 1
        return value
    }

    function peek() {
        if this.top == null {
            throw StackEmptyError
        }
        return this.top.data
    }

    function isEmpty() {
        return this.top == null
    }

    function size() {
        return this.size
    }
}
```

## Classic Algorithms

### Balanced Parentheses

```pseudocode
function isBalanced(expression) {
    stack = new Stack()
    pairs = { ')': '(', ']': '[', '}': '{' }

    for char in expression {
        if char in ['(', '[', '{'] {
            stack.push(char)
        } else if char in [')', ']', '}'] {
            if stack.isEmpty() {
                return false
            }
            if stack.pop() != pairs[char] {
                return false
            }
        }
    }

    return stack.isEmpty()
}
```

### Reverse Polish Notation (Postfix)

```pseudocode
function evaluateRPN(tokens) {
    stack = new Stack()

    for token in tokens {
        if isNumber(token) {
            stack.push(parseNumber(token))
        } else {
            b = stack.pop()
            a = stack.pop()

            switch token {
                case '+': stack.push(a + b)
                case '-': stack.push(a - b)
                case '*': stack.push(a * b)
                case '/': stack.push(a / b)
            }
        }
    }

    return stack.pop()
}
```

### Infix to Postfix (Shunting Yard)

```pseudocode
function infixToPostfix(tokens) {
    output = []
    operators = new Stack()
    precedence = { '+': 1, '-': 1, '*': 2, '/': 2, '^': 3 }

    for token in tokens {
        if isNumber(token) {
            output.append(token)
        } else if token == '(' {
            operators.push(token)
        } else if token == ')' {
            while operators.peek() != '(' {
                output.append(operators.pop())
            }
            operators.pop()  // Discard '('
        } else {
            while !operators.isEmpty()
                  and operators.peek() != '('
                  and precedence[operators.peek()] >= precedence[token] {
                output.append(operators.pop())
            }
            operators.push(token)
        }
    }

    while !operators.isEmpty() {
        output.append(operators.pop())
    }

    return output
}
```

## Use Cases

- **Function call stack**: Track return addresses and local variables
- **Undo operations**: Each action pushed, undo pops
- **Expression evaluation**: Parsing and evaluating mathematical expressions
- **Backtracking**: DFS, maze solving, puzzle solving
- **Browser history**: Back button uses stack
- **Syntax parsing**: Compilers check balanced brackets, tags
- **String reversal**: Push chars, pop in reverse order

## Advantages

- **Simple interface**: Only 3 main operations
- **Constant time operations**: O(1) for push, pop, peek
- **Memory efficient**: Array-based has minimal overhead
- **Natural for recursion**: Mimics call stack behavior
- **Easy to implement**: Straightforward logic

## Disadvantages

- **Limited access**: Can only access top element
- **No random access**: Must pop to reach lower elements
- **Fixed capacity** (array-based): May need resizing
- **Stack overflow**: Limited depth in recursive scenarios
- **No iteration**: Traversal requires popping (destructive)

## Comparison with Alternatives

| Aspect          | Stack        | Queue         | Deque        |
|-----------------|--------------|---------------|--------------|
| Access pattern  | LIFO         | FIFO          | Both ends    |
| Push/Pop        | One end      | Opposite ends | Both ends    |
| Use case        | Backtracking | Scheduling    | Flexible     |
| Complexity      | Simple       | Simple        | Moderate     |

## Common Pitfalls

- **Stack overflow**: Pushing too many elements (especially in recursion)
- **Stack underflow**: Popping from empty stack
- **Forgetting to check empty**: Always verify before pop/peek
- **Memory leaks**: Array-based should null popped indices
- **Incorrect capacity**: Array-based may resize unexpectedly
- **Confusing with queue**: LIFO vs FIFO mix-ups

## Related Structures

- **Queue**: FIFO instead of LIFO
- **Deque**: Double-ended, can act as stack
- **Min/Max Stack**: Stack that also tracks min/max in O(1)
- **Two-Stack Queue**: Queue implemented with two stacks
- **Call Stack**: Runtime stack for function execution

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
