# Heap (Binary Heap)

## Overview

A heap is a specialized complete binary tree that satisfies the heap property: in a max-heap, every parent is greater than or equal to its children; in a min-heap, every parent is less than or equal to its children. Heaps are commonly used to implement priority queues, offering O(log n) insertion and O(1) access to the maximum (or minimum) element.

## Properties

- **Complete binary tree**: All levels filled except possibly last, filled left to right
- **Heap property**: Parent relates to children by ordering (max or min)
- **Array representation**: Efficient storage without explicit pointers
- **Shape property**: Height is always floor(log n)
- **Not sorted**: Only partial ordering between parent and children

### Array Index Relationships

For 0-indexed array:
```
Parent(i) = (i - 1) / 2
LeftChild(i) = 2 * i + 1
RightChild(i) = 2 * i + 2
```

For 1-indexed array:
```
Parent(i) = i / 2
LeftChild(i) = 2 * i
RightChild(i) = 2 * i + 1
```

## Time Complexity

| Operation    | Average     | Worst       |
|--------------|-------------|-------------|
| Find max/min | O(1)        | O(1)        |
| Insert       | O(log n)    | O(log n)    |
| Extract max  | O(log n)    | O(log n)    |
| Delete       | O(log n)    | O(log n)    |
| Build heap   | O(n)        | O(n)        |
| Heapify      | O(log n)    | O(log n)    |

## Space Complexity

O(n) for n elements. Array-based implementation has no pointer overhead.

## Operations

### Helper Functions

```pseudocode
function parent(i) {
    return (i - 1) / 2
}

function leftChild(i) {
    return 2 * i + 1
}

function rightChild(i) {
    return 2 * i + 2
}

function swap(heap, i, j) {
    temp = heap[i]
    heap[i] = heap[j]
    heap[j] = temp
}
```

### Heapify Down (Sift Down)

Restore heap property by moving element down:

```pseudocode
// Max-heap version
function heapifyDown(heap, size, i) {
    largest = i
    left = leftChild(i)
    right = rightChild(i)

    if left < size and heap[left] > heap[largest] {
        largest = left
    }

    if right < size and heap[right] > heap[largest] {
        largest = right
    }

    if largest != i {
        swap(heap, i, largest)
        heapifyDown(heap, size, largest)
    }
}

// Min-heap version
function heapifyDownMin(heap, size, i) {
    smallest = i
    left = leftChild(i)
    right = rightChild(i)

    if left < size and heap[left] < heap[smallest] {
        smallest = left
    }

    if right < size and heap[right] < heap[smallest] {
        smallest = right
    }

    if smallest != i {
        swap(heap, i, smallest)
        heapifyDownMin(heap, size, smallest)
    }
}
```

### Heapify Up (Sift Up)

Restore heap property by moving element up:

```pseudocode
// Max-heap version
function heapifyUp(heap, i) {
    while i > 0 and heap[parent(i)] < heap[i] {
        swap(heap, i, parent(i))
        i = parent(i)
    }
}

// Min-heap version
function heapifyUpMin(heap, i) {
    while i > 0 and heap[parent(i)] > heap[i] {
        swap(heap, i, parent(i))
        i = parent(i)
    }
}
```

### Insert

```pseudocode
function insert(heap, value) {
    heap.append(value)
    heapifyUp(heap, heap.size - 1)
}
```

### Extract Max/Min

```pseudocode
// Max-heap
function extractMax(heap) {
    if heap.size == 0 {
        throw HeapEmptyError
    }

    max = heap[0]
    heap[0] = heap[heap.size - 1]
    heap.removeLast()

    if heap.size > 0 {
        heapifyDown(heap, heap.size, 0)
    }

    return max
}

// Min-heap
function extractMin(heap) {
    if heap.size == 0 {
        throw HeapEmptyError
    }

    min = heap[0]
    heap[0] = heap[heap.size - 1]
    heap.removeLast()

    if heap.size > 0 {
        heapifyDownMin(heap, heap.size, 0)
    }

    return min
}
```

### Build Heap

Build heap from unordered array in O(n):

```pseudocode
function buildHeap(array) {
    n = array.length

    // Start from last non-leaf node
    for i from (n / 2) - 1 down to 0 {
        heapifyDown(array, n, i)
    }

    return array
}
```

### Peek

```pseudocode
function peek(heap) {
    if heap.size == 0 {
        throw HeapEmptyError
    }
    return heap[0]
}
```

## Implementation

### Max Heap

```pseudocode
class MaxHeap {
    data = []

    function insert(value) {
        this.data.append(value)
        this.heapifyUp(this.data.length - 1)
    }

    function extractMax() {
        if this.data.length == 0 {
            throw HeapEmptyError
        }

        max = this.data[0]
        last = this.data.pop()

        if this.data.length > 0 {
            this.data[0] = last
            this.heapifyDown(0)
        }

        return max
    }

    function peek() {
        if this.data.length == 0 {
            throw HeapEmptyError
        }
        return this.data[0]
    }

    function heapifyUp(i) {
        while i > 0 {
            p = (i - 1) / 2
            if this.data[p] >= this.data[i] {
                break
            }
            swap(this.data, p, i)
            i = p
        }
    }

    function heapifyDown(i) {
        n = this.data.length

        while true {
            largest = i
            left = 2 * i + 1
            right = 2 * i + 2

            if left < n and this.data[left] > this.data[largest] {
                largest = left
            }
            if right < n and this.data[right] > this.data[largest] {
                largest = right
            }

            if largest == i {
                break
            }

            swap(this.data, i, largest)
            i = largest
        }
    }

    function size() {
        return this.data.length
    }

    function isEmpty() {
        return this.data.length == 0
    }
}
```

### Min Heap

```pseudocode
class MinHeap {
    data = []

    function insert(value) {
        this.data.append(value)
        this.heapifyUp(this.data.length - 1)
    }

    function extractMin() {
        if this.data.length == 0 {
            throw HeapEmptyError
        }

        min = this.data[0]
        last = this.data.pop()

        if this.data.length > 0 {
            this.data[0] = last
            this.heapifyDown(0)
        }

        return min
    }

    function peek() {
        if this.data.length == 0 {
            throw HeapEmptyError
        }
        return this.data[0]
    }

    function heapifyUp(i) {
        while i > 0 {
            p = (i - 1) / 2
            if this.data[p] <= this.data[i] {
                break
            }
            swap(this.data, p, i)
            i = p
        }
    }

    function heapifyDown(i) {
        n = this.data.length

        while true {
            smallest = i
            left = 2 * i + 1
            right = 2 * i + 2

            if left < n and this.data[left] < this.data[smallest] {
                smallest = left
            }
            if right < n and this.data[right] < this.data[smallest] {
                smallest = right
            }

            if smallest == i {
                break
            }

            swap(this.data, i, smallest)
            i = smallest
        }
    }

    function size() {
        return this.data.length
    }

    function isEmpty() {
        return this.data.length == 0
    }
}
```

### Priority Queue

```pseudocode
class PriorityQueue {
    heap = new MinHeap()  // Or MaxHeap depending on priority ordering

    function enqueue(item, priority) {
        this.heap.insert({ item: item, priority: priority })
    }

    function dequeue() {
        entry = this.heap.extractMin()
        return entry.item
    }

    function peek() {
        entry = this.heap.peek()
        return entry.item
    }

    function isEmpty() {
        return this.heap.isEmpty()
    }

    function size() {
        return this.heap.size()
    }
}
```

## Classic Algorithms

### Heap Sort

```pseudocode
function heapSort(array) {
    n = array.length

    // Build max heap
    for i from n / 2 - 1 down to 0 {
        heapifyDown(array, n, i)
    }

    // Extract elements one by one
    for i from n - 1 down to 1 {
        swap(array, 0, i)
        heapifyDown(array, i, 0)
    }

    return array
}
```

### K Largest Elements

```pseudocode
function kLargest(array, k) {
    // Use min-heap of size k
    minHeap = new MinHeap()

    for value in array {
        if minHeap.size() < k {
            minHeap.insert(value)
        } else if value > minHeap.peek() {
            minHeap.extractMin()
            minHeap.insert(value)
        }
    }

    return minHeap.data
}
```

### Merge K Sorted Lists

```pseudocode
function mergeKSortedLists(lists) {
    minHeap = new MinHeap()  // Stores (value, listIndex, elementIndex)
    result = []

    // Initialize heap with first element from each list
    for i from 0 to lists.length - 1 {
        if lists[i].length > 0 {
            minHeap.insert((lists[i][0], i, 0))
        }
    }

    while not minHeap.isEmpty() {
        (value, listIdx, elemIdx) = minHeap.extractMin()
        result.append(value)

        // Add next element from same list
        if elemIdx + 1 < lists[listIdx].length {
            minHeap.insert((lists[listIdx][elemIdx + 1], listIdx, elemIdx + 1))
        }
    }

    return result
}
```

### Find Median in Stream

```pseudocode
class MedianFinder {
    maxHeap = new MaxHeap()  // Lower half
    minHeap = new MinHeap()  // Upper half

    function addNum(num) {
        // Add to max heap (lower half)
        maxHeap.insert(num)

        // Balance: move largest from lower to upper
        minHeap.insert(maxHeap.extractMax())

        // Ensure lower half >= upper half in size
        if maxHeap.size() < minHeap.size() {
            maxHeap.insert(minHeap.extractMin())
        }
    }

    function findMedian() {
        if maxHeap.size() > minHeap.size() {
            return maxHeap.peek()
        }
        return (maxHeap.peek() + minHeap.peek()) / 2.0
    }
}
```

## Use Cases

- **Priority queues**: Task scheduling, event simulation
- **Heap sort**: In-place O(n log n) sorting
- **Selection algorithms**: Finding k largest/smallest elements
- **Graph algorithms**: Dijkstra's, Prim's MST
- **Median maintenance**: Running median in data streams
- **Merge operations**: Merging sorted sequences
- **OS scheduling**: Process priority management

## Advantages

- **O(1) access to max/min**: Instant access to extreme element
- **Efficient modifications**: O(log n) insert and extract
- **Array-based**: Cache-friendly, no pointer overhead
- **In-place**: Heap sort needs no extra space
- **Build in O(n)**: Linear time heap construction

## Disadvantages

- **Not sorted**: Only partial ordering
- **No efficient search**: O(n) to find arbitrary element
- **No decrease-key**: Standard heap doesn't support efficiently
- **Only one extreme**: Access either max or min, not both
- **Unstable**: Equal elements may change order

## Comparison with Alternatives

| Aspect          | Binary Heap | BST         | Sorted Array | Fibonacci Heap |
|-----------------|-------------|-------------|--------------|----------------|
| Find max/min    | O(1)        | O(log n)    | O(1)         | O(1)           |
| Insert          | O(log n)    | O(log n)    | O(n)         | O(1)*          |
| Extract         | O(log n)    | O(log n)    | O(1)         | O(log n)*      |
| Decrease key    | O(n)        | O(log n)    | O(n)         | O(1)*          |
| Build           | O(n)        | O(n log n)  | O(n log n)   | O(n)           |
| Space           | O(n)        | O(n)        | O(n)         | O(n)           |

*Amortized

## Common Pitfalls

- **Index confusion**: 0-indexed vs 1-indexed formulas
- **Off-by-one**: In heapify range calculations
- **Wrong heap type**: Using max-heap when min-heap needed
- **Forgetting to heapify**: After direct array modifications
- **Build heap direction**: Must go bottom-up, not top-down
- **Heap property scope**: Only parent-child, not siblings

## Related Structures

- **Fibonacci Heap**: Better amortized bounds for decrease-key
- **Binomial Heap**: Mergeable heap
- **Pairing Heap**: Simpler alternative to Fibonacci heap
- **d-ary Heap**: Generalization with d children
- **Min-Max Heap**: Access both min and max in O(1)
- **Treap**: BST + heap hybrid using random priorities

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
