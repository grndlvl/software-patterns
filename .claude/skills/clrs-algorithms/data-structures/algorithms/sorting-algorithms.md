# Sorting Algorithms

## Overview

Sorting is the process of arranging elements in a specific order (typically ascending or descending). It's one of the most fundamental operations in computer science, serving as a building block for many other algorithms and appearing in countless real-world applications.

### The Sorting Problem

**Input**: An array A of n elements with a total ordering relation
**Output**: A permutation of A such that A[0] ≤ A[1] ≤ ... ≤ A[n-1]

### Stability

A sorting algorithm is **stable** if it preserves the relative order of equal elements.

**Example**:
```
Input:  [(3, "a"), (1, "b"), (3, "c"), (2, "d")]
Stable:   [(1, "b"), (2, "d"), (3, "a"), (3, "c")]  // "a" still before "c"
Unstable: [(1, "b"), (2, "d"), (3, "c"), (3, "a")]  // "c" before "a"
```

**Why stability matters**: Multi-level sorting (sort by name, then by date), database operations, maintaining insertion order for equal keys.

## Complexity Comparison

| Algorithm      | Best        | Average     | Worst       | Space       | Stable | In-place | Notes                    |
|----------------|-------------|-------------|-------------|-------------|--------|----------|--------------------------|
| QuickSort      | O(n log n)  | O(n log n)  | O(n²)       | O(log n)    | No     | Yes      | Fastest average case     |
| MergeSort      | O(n log n)  | O(n log n)  | O(n log n)  | O(n)        | Yes    | No       | Guaranteed O(n log n)    |
| HeapSort       | O(n log n)  | O(n log n)  | O(n log n)  | O(1)        | No     | Yes      | In-place, guaranteed     |
| TimSort        | O(n)        | O(n log n)  | O(n log n)  | O(n)        | Yes    | No       | Python/Java default      |
| InsertionSort  | O(n)        | O(n²)       | O(n²)       | O(1)        | Yes    | Yes      | Best for small/sorted    |
| ShellSort      | O(n log n)  | O(n^1.5)    | O(n²)       | O(1)        | No     | Yes      | Gap-based insertion      |
| SelectionSort  | O(n²)       | O(n²)       | O(n²)       | O(1)        | No     | Yes      | Never use in production  |
| BubbleSort     | O(n)        | O(n²)       | O(n²)       | O(1)        | Yes    | Yes      | Educational only         |
| CountingSort   | O(n+k)      | O(n+k)      | O(n+k)      | O(k)        | Yes    | No       | Limited to small ranges  |
| RadixSort      | O(d(n+k))   | O(d(n+k))   | O(d(n+k))   | O(n+k)      | Yes    | No       | Fixed-length keys        |
| BucketSort     | O(n+k)      | O(n+k)      | O(n²)       | O(n+k)      | Yes    | No       | Uniform distribution     |

**Legend**: k = range of values, d = number of digits

## Comparison-Based Sorts

These algorithms make decisions based solely on comparing elements. Theoretical lower bound: O(n log n).

### 1. QuickSort

**Overview**: Divide-and-conquer algorithm that partitions array around a pivot, recursively sorting sub-arrays. Fastest average-case comparison sort.

**Time Complexity**:
- Best/Average: O(n log n)
- Worst: O(n²) (rare with randomization)

**Space**: O(log n) for recursion stack

**Properties**: Unstable, in-place, cache-friendly

#### Basic QuickSort (Lomuto Partition)

```pseudocode
function quickSort(array, low, high) {
    if low < high {
        pivotIndex = partition(array, low, high)
        quickSort(array, low, pivotIndex - 1)
        quickSort(array, pivotIndex + 1, high)
    }
}

function partition(array, low, high) {
    // Choose last element as pivot
    pivot = array[high]
    i = low - 1  // Index of smaller element

    for j from low to high - 1 {
        if array[j] <= pivot {
            i++
            swap(array, i, j)
        }
    }

    swap(array, i + 1, high)
    return i + 1
}

function swap(array, i, j) {
    temp = array[i]
    array[i] = array[j]
    array[j] = temp
}
```

#### Hoare Partition Scheme

More efficient, fewer swaps:

```pseudocode
function partitionHoare(array, low, high) {
    pivot = array[low]  // First element as pivot
    i = low - 1
    j = high + 1

    while true {
        do {
            i++
        } while array[i] < pivot

        do {
            j--
        } while array[j] > pivot

        if i >= j {
            return j
        }

        swap(array, i, j)
    }
}

function quickSortHoare(array, low, high) {
    if low < high {
        p = partitionHoare(array, low, high)
        quickSortHoare(array, low, p)
        quickSortHoare(array, p + 1, high)
    }
}
```

#### Randomized QuickSort

Avoids worst-case on sorted/reverse-sorted inputs:

```pseudocode
function randomizedPartition(array, low, high) {
    // Random pivot selection
    randomIndex = random(low, high)
    swap(array, randomIndex, high)
    return partition(array, low, high)
}

function randomizedQuickSort(array, low, high) {
    if low < high {
        pivotIndex = randomizedPartition(array, low, high)
        randomizedQuickSort(array, low, pivotIndex - 1)
        randomizedQuickSort(array, pivotIndex + 1, high)
    }
}
```

#### Three-Way Partition (Dutch National Flag)

Optimal for arrays with many duplicates:

```pseudocode
function quickSort3Way(array, low, high) {
    if low >= high {
        return
    }

    lt = low          // array[low..lt-1] < pivot
    gt = high         // array[gt+1..high] > pivot
    i = low + 1       // array[lt..i-1] == pivot
    pivot = array[low]

    while i <= gt {
        if array[i] < pivot {
            swap(array, lt, i)
            lt++
            i++
        } else if array[i] > pivot {
            swap(array, i, gt)
            gt--
        } else {
            i++
        }
    }

    quickSort3Way(array, low, lt - 1)
    quickSort3Way(array, gt + 1, high)
}
```

#### Median-of-Three Pivot Selection

Better pivot choice for partially sorted data:

```pseudocode
function medianOfThree(array, low, high) {
    mid = low + (high - low) / 2

    // Sort low, mid, high
    if array[mid] < array[low] {
        swap(array, low, mid)
    }
    if array[high] < array[low] {
        swap(array, low, high)
    }
    if array[mid] < array[high] {
        swap(array, mid, high)
    }

    // Place median at high-1
    swap(array, mid, high - 1)
    return array[high - 1]
}

function quickSortMedian(array, low, high) {
    if low + 10 <= high {  // Use insertion sort for small arrays
        pivot = medianOfThree(array, low, high)
        i = partition(array, low, high - 1)
        quickSortMedian(array, low, i - 1)
        quickSortMedian(array, i + 1, high)
    } else {
        insertionSort(array, low, high)
    }
}
```

### 2. MergeSort

**Overview**: Divide-and-conquer algorithm that divides array in half, recursively sorts halves, then merges sorted halves. Guaranteed O(n log n) performance.

**Time Complexity**: O(n log n) in all cases

**Space**: O(n) for auxiliary array

**Properties**: Stable, not in-place, excellent for linked lists

#### Top-Down Recursive MergeSort

```pseudocode
function mergeSort(array, left, right) {
    if left < right {
        mid = left + (right - left) / 2

        mergeSort(array, left, mid)
        mergeSort(array, mid + 1, right)
        merge(array, left, mid, right)
    }
}

function merge(array, left, mid, right) {
    // Create temporary arrays
    leftSize = mid - left + 1
    rightSize = right - mid

    leftArray = new Array[leftSize]
    rightArray = new Array[rightSize]

    // Copy data to temp arrays
    for i from 0 to leftSize - 1 {
        leftArray[i] = array[left + i]
    }
    for j from 0 to rightSize - 1 {
        rightArray[j] = array[mid + 1 + j]
    }

    // Merge temp arrays back
    i = 0  // Initial index of left subarray
    j = 0  // Initial index of right subarray
    k = left  // Initial index of merged array

    while i < leftSize and j < rightSize {
        if leftArray[i] <= rightArray[j] {
            array[k] = leftArray[i]
            i++
        } else {
            array[k] = rightArray[j]
            j++
        }
        k++
    }

    // Copy remaining elements
    while i < leftSize {
        array[k] = leftArray[i]
        i++
        k++
    }

    while j < rightSize {
        array[k] = rightArray[j]
        j++
        k++
    }
}
```

#### Bottom-Up Iterative MergeSort

No recursion, better for some applications:

```pseudocode
function mergeSortIterative(array) {
    n = array.length

    // Start with merge subarrays of size 1, double size each iteration
    for size from 1 to n - 1 step size * 2 {
        // Pick starting index of left sub array to be merged
        for leftStart from 0 to n - 1 step size * 2 {
            mid = min(leftStart + size - 1, n - 1)
            rightEnd = min(leftStart + size * 2 - 1, n - 1)

            if mid < rightEnd {
                merge(array, leftStart, mid, rightEnd)
            }
        }
    }
}
```

#### In-Place Merge (Advanced)

Reduces space to O(1) but increases time complexity:

```pseudocode
function mergeSortInPlace(array, left, right) {
    if left < right {
        mid = left + (right - left) / 2

        mergeSortInPlace(array, left, mid)
        mergeSortInPlace(array, mid + 1, right)
        mergeInPlace(array, left, mid, right)
    }
}

function mergeInPlace(array, left, mid, right) {
    start2 = mid + 1

    // If already sorted
    if array[mid] <= array[start2] {
        return
    }

    while left <= mid and start2 <= right {
        if array[left] <= array[start2] {
            left++
        } else {
            value = array[start2]
            index = start2

            // Shift elements
            while index != left {
                array[index] = array[index - 1]
                index--
            }
            array[left] = value

            left++
            mid++
            start2++
        }
    }
}
```

#### External Merge Sort

For data too large to fit in memory:

```pseudocode
function externalMergeSort(inputFile, outputFile, memorySize) {
    // Phase 1: Create sorted runs
    runs = []
    while not inputFile.eof() {
        chunk = inputFile.read(memorySize)
        inMemorySort(chunk)  // Use any in-memory sort
        runFile = writeToTempFile(chunk)
        runs.append(runFile)
    }

    // Phase 2: Merge runs
    while runs.length > 1 {
        newRuns = []
        for i from 0 to runs.length - 1 step 2 {
            if i + 1 < runs.length {
                merged = mergeTwoFiles(runs[i], runs[i + 1])
                newRuns.append(merged)
            } else {
                newRuns.append(runs[i])
            }
        }
        runs = newRuns
    }

    copyFile(runs[0], outputFile)
}
```

### 3. HeapSort

**Overview**: Build a max-heap, then repeatedly extract maximum to get sorted order. Combines best of merge sort (O(n log n) guarantee) and insertion sort (in-place).

**Time Complexity**: O(n log n) in all cases

**Space**: O(1)

**Properties**: Unstable, in-place, not cache-friendly

```pseudocode
function heapSort(array) {
    n = array.length

    // Build max heap (heapify)
    for i from n / 2 - 1 down to 0 {
        heapifyDown(array, n, i)
    }

    // Extract elements one by one from heap
    for i from n - 1 down to 1 {
        // Move current root to end
        swap(array, 0, i)

        // Heapify reduced heap
        heapifyDown(array, i, 0)
    }
}

function heapifyDown(array, heapSize, i) {
    largest = i
    left = 2 * i + 1
    right = 2 * i + 2

    if left < heapSize and array[left] > array[largest] {
        largest = left
    }

    if right < heapSize and array[right] > array[largest] {
        largest = right
    }

    if largest != i {
        swap(array, i, largest)
        heapifyDown(array, heapSize, largest)
    }
}
```

### 4. InsertionSort

**Overview**: Build sorted array one element at a time by inserting each element into its correct position. Excellent for small arrays and nearly sorted data.

**Time Complexity**:
- Best: O(n) - already sorted
- Average/Worst: O(n²)

**Space**: O(1)

**Properties**: Stable, in-place, adaptive, online

```pseudocode
function insertionSort(array) {
    n = array.length

    for i from 1 to n - 1 {
        key = array[i]
        j = i - 1

        // Move elements greater than key one position ahead
        while j >= 0 and array[j] > key {
            array[j + 1] = array[j]
            j--
        }

        array[j + 1] = key
    }
}

// Binary search variant - reduces comparisons
function binaryInsertionSort(array) {
    n = array.length

    for i from 1 to n - 1 {
        key = array[i]

        // Find insertion position using binary search
        pos = binarySearch(array, key, 0, i - 1)

        // Shift elements
        for j from i - 1 down to pos {
            array[j + 1] = array[j]
        }

        array[pos] = key
    }
}

function binarySearch(array, key, low, high) {
    while low <= high {
        mid = low + (high - low) / 2

        if array[mid] == key {
            return mid + 1
        } else if array[mid] < key {
            low = mid + 1
        } else {
            high = mid - 1
        }
    }
    return low
}
```

### 5. TimSort

**Overview**: Hybrid stable sorting algorithm derived from merge sort and insertion sort. Designed to perform well on real-world data. Default sort in Python and Java.

**Time Complexity**:
- Best: O(n) - already sorted runs
- Average/Worst: O(n log n)

**Space**: O(n)

**Properties**: Stable, adaptive, identifies existing runs

```pseudocode
const MIN_MERGE = 32

function timSort(array) {
    n = array.length
    minRun = calculateMinRun(n)

    // Sort individual runs using insertion sort
    for start from 0 to n - 1 step minRun {
        end = min(start + minRun - 1, n - 1)
        insertionSort(array, start, end)
    }

    // Merge sorted runs
    size = minRun
    while size < n {
        for start from 0 to n - 1 step size * 2 {
            mid = start + size - 1
            end = min(start + size * 2 - 1, n - 1)

            if mid < end {
                merge(array, start, mid, end)
            }
        }
        size = size * 2
    }
}

function calculateMinRun(n) {
    // Returns minimum run length such that n/minRun is power of 2
    // or close to it, between 32 and 64
    r = 0
    while n >= MIN_MERGE {
        r = r | (n & 1)
        n = n >> 1
    }
    return n + r
}

// Enhanced insertion sort for small runs
function insertionSort(array, left, right) {
    for i from left + 1 to right {
        key = array[i]
        j = i - 1

        while j >= left and array[j] > key {
            array[j + 1] = array[j]
            j--
        }
        array[j + 1] = key
    }
}

// Galloping mode for merging runs (advanced optimization)
function gallopingMerge(array, left, mid, right) {
    // Use galloping search when one run is winning consistently
    // Details omitted for brevity - see Python's timsort.txt
}
```

### 6. ShellSort

**Overview**: Generalization of insertion sort that allows exchange of far-apart elements. Uses gap sequences.

**Time Complexity**: Depends on gap sequence
- Best known: O(n log² n)
- Average: O(n^1.5) with good gaps

**Space**: O(1)

**Properties**: Unstable, in-place

```pseudocode
function shellSort(array) {
    n = array.length

    // Start with large gap, reduce each iteration
    gap = n / 2

    while gap > 0 {
        // Gapped insertion sort
        for i from gap to n - 1 {
            temp = array[i]
            j = i

            while j >= gap and array[j - gap] > temp {
                array[j] = array[j - gap]
                j -= gap
            }

            array[j] = temp
        }

        gap = gap / 2
    }
}

// Using Knuth's gap sequence: h = 3h + 1
function shellSortKnuth(array) {
    n = array.length

    // Find largest gap in sequence
    gap = 1
    while gap < n / 3 {
        gap = 3 * gap + 1
    }

    while gap >= 1 {
        for i from gap to n - 1 {
            temp = array[i]
            j = i

            while j >= gap and array[j - gap] > temp {
                array[j] = array[j - gap]
                j -= gap
            }

            array[j] = temp
        }

        gap = gap / 3
    }
}
```

## Non-Comparison Sorts

These algorithms exploit specific properties of keys to sort faster than O(n log n).

### 7. Counting Sort

**Overview**: Count occurrences of each value, then reconstruct sorted array. Works when range of values is known and relatively small.

**Time Complexity**: O(n + k) where k is range

**Space**: O(k)

**Properties**: Stable, not in-place

**Constraint**: Integer keys in known range

```pseudocode
function countingSort(array, maxValue) {
    n = array.length
    count = new Array[maxValue + 1] filled with 0
    output = new Array[n]

    // Store count of each element
    for i from 0 to n - 1 {
        count[array[i]]++
    }

    // Change count[i] to contain actual position
    for i from 1 to maxValue {
        count[i] += count[i - 1]
    }

    // Build output array (iterate backwards for stability)
    for i from n - 1 down to 0 {
        output[count[array[i]] - 1] = array[i]
        count[array[i]]--
    }

    // Copy output to original array
    for i from 0 to n - 1 {
        array[i] = output[i]
    }
}

// Handle negative numbers
function countingSortNegatives(array) {
    if array.length == 0 {
        return
    }

    // Find min and max
    minVal = min(array)
    maxVal = max(array)
    range = maxVal - minVal + 1

    count = new Array[range] filled with 0
    output = new Array[array.length]

    // Count occurrences (offset by minVal)
    for i from 0 to array.length - 1 {
        count[array[i] - minVal]++
    }

    // Cumulative count
    for i from 1 to range - 1 {
        count[i] += count[i - 1]
    }

    // Build output
    for i from array.length - 1 down to 0 {
        index = array[i] - minVal
        output[count[index] - 1] = array[i]
        count[index]--
    }

    // Copy back
    for i from 0 to array.length - 1 {
        array[i] = output[i]
    }
}
```

### 8. Radix Sort

**Overview**: Sort by processing individual digits/characters from least significant to most significant (or vice versa). Uses stable sort (usually counting sort) as subroutine.

**Time Complexity**: O(d(n + k)) where d is digits, k is base

**Space**: O(n + k)

**Properties**: Stable, not in-place

**Constraint**: Fixed-length keys

```pseudocode
function radixSort(array) {
    // Find maximum to determine number of digits
    maxVal = max(array)

    // Do counting sort for every digit
    exp = 1  // Current digit position (1s, 10s, 100s, ...)
    while maxVal / exp > 0 {
        countingSortByDigit(array, exp)
        exp *= 10
    }
}

function countingSortByDigit(array, exp) {
    n = array.length
    output = new Array[n]
    count = new Array[10] filled with 0  // Base 10

    // Store count of occurrences
    for i from 0 to n - 1 {
        digit = (array[i] / exp) % 10
        count[digit]++
    }

    // Cumulative count
    for i from 1 to 9 {
        count[i] += count[i - 1]
    }

    // Build output array (backwards for stability)
    for i from n - 1 down to 0 {
        digit = (array[i] / exp) % 10
        output[count[digit] - 1] = array[i]
        count[digit]--
    }

    // Copy to original array
    for i from 0 to n - 1 {
        array[i] = output[i]
    }
}

// Most Significant Digit (MSD) radix sort
function msdRadixSort(array, left, right, digit) {
    if left >= right or digit < 0 {
        return
    }

    const BASE = 256  // For byte-wise sorting
    count = new Array[BASE + 1] filled with 0

    // Count frequencies
    for i from left to right {
        d = getDigit(array[i], digit)
        count[d + 1]++
    }

    // Compute cumulative counts
    for i from 1 to BASE {
        count[i] += count[i - 1]
    }

    // Distribute
    aux = new Array[right - left + 1]
    for i from left to right {
        d = getDigit(array[i], digit)
        aux[count[d]++] = array[i]
    }

    // Copy back
    for i from left to right {
        array[i] = aux[i - left]
    }

    // Recursively sort each bucket
    for i from 0 to BASE - 1 {
        msdRadixSort(array, left + count[i], left + count[i + 1] - 1, digit - 1)
    }
}
```

### 9. Bucket Sort

**Overview**: Distribute elements into buckets, sort buckets individually, then concatenate. Works best when input is uniformly distributed.

**Time Complexity**:
- Best/Average: O(n + k)
- Worst: O(n²) if all in one bucket

**Space**: O(n + k)

**Properties**: Stable (if using stable bucket sort), not in-place

```pseudocode
function bucketSort(array, numBuckets) {
    if array.length == 0 {
        return
    }

    // Find min and max
    minVal = min(array)
    maxVal = max(array)
    range = maxVal - minVal

    // Create buckets
    buckets = new Array[numBuckets]
    for i from 0 to numBuckets - 1 {
        buckets[i] = new List()
    }

    // Distribute elements into buckets
    for i from 0 to array.length - 1 {
        // Map value to bucket index
        bucketIndex = (array[i] - minVal) * numBuckets / (range + 1)
        buckets[bucketIndex].append(array[i])
    }

    // Sort individual buckets and concatenate
    index = 0
    for i from 0 to numBuckets - 1 {
        insertionSort(buckets[i])  // Or any other sort

        for j from 0 to buckets[i].length - 1 {
            array[index] = buckets[i][j]
            index++
        }
    }
}

// Bucket sort for floating point in [0, 1)
function bucketSortFloat(array) {
    n = array.length
    buckets = new Array[n]

    for i from 0 to n - 1 {
        buckets[i] = new List()
    }

    // Put elements in buckets
    for i from 0 to n - 1 {
        bucketIndex = floor(n * array[i])
        buckets[bucketIndex].append(array[i])
    }

    // Sort each bucket
    for i from 0 to n - 1 {
        insertionSort(buckets[i])
    }

    // Concatenate buckets
    index = 0
    for i from 0 to n - 1 {
        for j from 0 to buckets[i].length - 1 {
            array[index] = buckets[i][j]
            index++
        }
    }
}
```

## Algorithm Selection Guide

### When to Use Each Algorithm

| Scenario | Best Choice | Reason |
|----------|-------------|--------|
| General purpose, unknown data | **QuickSort** (randomized) | Fastest average case |
| Need stability | **MergeSort** or **TimSort** | Guaranteed stable |
| Limited memory (in-place required) | **HeapSort** | O(1) space, O(n log n) guaranteed |
| Small arrays (n < 50) | **InsertionSort** | Low overhead, cache-friendly |
| Nearly sorted data | **InsertionSort** or **TimSort** | O(n) on sorted, adaptive |
| Guaranteed O(n log n), no recursion | **HeapSort** | No stack overflow risk |
| Linked lists | **MergeSort** | Natural for linked structures |
| External sorting (disk) | **External MergeSort** | Minimizes I/O operations |
| Many duplicate values | **3-way QuickSort** | O(n) with all equal |
| Integer keys, small range | **Counting Sort** | Linear time |
| Fixed-length strings/integers | **Radix Sort** | Faster than O(n log n) |
| Uniform distribution in range | **Bucket Sort** | Linear expected time |
| Real-world mixed data | **TimSort** | Adaptive to patterns |
| Parallel processing | **MergeSort** | Easily parallelizable |
| Teaching/learning | **QuickSort** or **MergeSort** | Clear divide-and-conquer |

### Hybrid Approaches

Modern implementations combine algorithms:

```pseudocode
function hybridSort(array, left, right) {
    // Use insertion sort for small subarrays
    if right - left + 1 < INSERTION_THRESHOLD {
        insertionSort(array, left, right)
        return
    }

    // Check if nearly sorted
    if isNearlySorted(array, left, right) {
        insertionSort(array, left, right)
        return
    }

    // Use quicksort with fallback to heapsort
    depth = 0
    maxDepth = 2 * log2(right - left + 1)
    introsort(array, left, right, depth, maxDepth)
}

// Introspective sort (C++ std::sort)
function introsort(array, left, right, depth, maxDepth) {
    if right - left + 1 < INSERTION_THRESHOLD {
        insertionSort(array, left, right)
        return
    }

    // Switch to heapsort if recursion too deep
    if depth > maxDepth {
        heapSort(array, left, right)
        return
    }

    // Otherwise use quicksort
    pivot = partition(array, left, right)
    introsort(array, left, pivot - 1, depth + 1, maxDepth)
    introsort(array, pivot + 1, right, depth + 1, maxDepth)
}
```

## In-Place vs Stable Trade-offs

**Cannot have both in-place AND stable with O(n log n) comparison sort** (open problem).

### Trade-off Matrix

| Property | QuickSort | MergeSort | HeapSort | TimSort |
|----------|-----------|-----------|----------|---------|
| In-place | ✓ | ✗ | ✓ | ✗ |
| Stable | ✗ | ✓ | ✗ | ✓ |
| O(n log n) worst | ✗ | ✓ | ✓ | ✓ |
| Cache-friendly | ✓ | ~ | ✗ | ✓ |
| Simple implementation | ✓ | ✓ | ~ | ✗ |

### Making Unstable Sorts Stable

Attach original index to each element:

```pseudocode
function stableWrapperSort(array) {
    // Create pairs (value, originalIndex)
    pairs = []
    for i from 0 to array.length - 1 {
        pairs.append((array[i], i))
    }

    // Sort by value, break ties by index
    unstableSort(pairs, comparePairs)

    // Extract values
    for i from 0 to array.length - 1 {
        array[i] = pairs[i].value
    }
}

function comparePairs(a, b) {
    if a.value != b.value {
        return a.value - b.value
    }
    return a.index - b.index  // Preserve original order
}
```

## Parallel Sorting Considerations

### Parallel MergeSort

```pseudocode
function parallelMergeSort(array, left, right, depth, maxDepth) {
    if left >= right {
        return
    }

    mid = left + (right - left) / 2

    // Spawn parallel tasks if not too deep
    if depth < maxDepth {
        task1 = async parallelMergeSort(array, left, mid, depth + 1, maxDepth)
        task2 = async parallelMergeSort(array, mid + 1, right, depth + 1, maxDepth)
        await task1
        await task2
    } else {
        // Sequential for small subproblems
        mergeSort(array, left, mid)
        mergeSort(array, mid + 1, right)
    }

    merge(array, left, mid, right)
}
```

### Parallel QuickSort

```pseudocode
function parallelQuickSort(array, left, right, depth, maxDepth) {
    if left >= right {
        return
    }

    pivot = partition(array, left, right)

    if depth < maxDepth {
        task1 = async parallelQuickSort(array, left, pivot - 1, depth + 1, maxDepth)
        task2 = async parallelQuickSort(array, pivot + 1, right, depth + 1, maxDepth)
        await task1
        await task2
    } else {
        quickSort(array, left, pivot - 1)
        quickSort(array, pivot + 1, right)
    }
}
```

## Common Pitfalls

### 1. QuickSort Pitfalls

```pseudocode
// WRONG: Always choosing first/last element as pivot
function badQuickSort(array, low, high) {
    pivot = array[high]  // Predictable - O(n²) on sorted data
    ...
}

// RIGHT: Randomize or use median-of-three
function goodQuickSort(array, low, high) {
    randomIndex = random(low, high)
    swap(array, randomIndex, high)
    pivot = array[high]
    ...
}
```

### 2. MergeSort Pitfalls

```pseudocode
// WRONG: Creating new arrays in every merge call
function wastefulMerge(array, left, mid, right) {
    leftArray = []  // Memory allocation in hot loop
    rightArray = []
    ...
}

// RIGHT: Reuse auxiliary array
auxArray = new Array[array.length]

function efficientMerge(array, aux, left, mid, right) {
    // Use pre-allocated aux
    ...
}
```

### 3. Stability Pitfalls

```pseudocode
// WRONG: Non-stable comparison in supposedly stable sort
if array[i] > key {  // Strict inequality
    ...
}

// RIGHT: Maintain stability
if array[i] <= key {  // Use <= to preserve order of equals
    ...
}
```

### 4. Integer Overflow

```pseudocode
// WRONG: Can overflow on large indices
mid = (left + right) / 2

// RIGHT: Prevents overflow
mid = left + (right - left) / 2
```

### 5. Counting Sort Range Issues

```pseudocode
// WRONG: Assuming range starts at 0
count = new Array[max(array)]  // Fails if min is negative or large

// RIGHT: Account for actual range
min = min(array)
max = max(array)
count = new Array[max - min + 1]
```

### 6. Off-by-One Errors

```pseudocode
// WRONG: Loop bounds
for i from 0 to n {  // Goes one past end
    ...
}

// RIGHT
for i from 0 to n - 1 {
    ...
}
```

### 7. Not Checking Empty/Single Element

```pseudocode
// WRONG: No base case check
function sort(array) {
    mid = array.length / 2  // Crashes on empty array
    ...
}

// RIGHT: Guard clauses
function sort(array) {
    if array.length <= 1 {
        return
    }
    ...
}
```

## Performance Optimization Tips

### 1. Insertion Sort Threshold

```pseudocode
const INSERTION_THRESHOLD = 10  // Tune based on hardware

function optimizedQuickSort(array, left, right) {
    if right - left + 1 < INSERTION_THRESHOLD {
        insertionSort(array, left, right)
        return
    }

    pivot = partition(array, left, right)
    optimizedQuickSort(array, left, pivot - 1)
    optimizedQuickSort(array, pivot + 1, right)
}
```

### 2. Tail Recursion Elimination

```pseudocode
function tailOptimizedQuickSort(array, left, right) {
    while left < right {
        pivot = partition(array, left, right)

        // Recurse on smaller partition, iterate on larger
        if pivot - left < right - pivot {
            tailOptimizedQuickSort(array, left, pivot - 1)
            left = pivot + 1  // Tail recursion → iteration
        } else {
            tailOptimizedQuickSort(array, pivot + 1, right)
            right = pivot - 1
        }
    }
}
```

### 3. Cache-Friendly Access Patterns

```pseudocode
// Prefer sequential access over random access
// Good: Insertion sort (sequential)
// Bad: HeapSort (random access pattern)
```

### 4. Minimize Comparisons

```pseudocode
// Use binary insertion sort for small arrays
// when comparisons are expensive
function binaryInsertionSort(array) {
    for i from 1 to n - 1 {
        key = array[i]
        pos = binarySearch(array, 0, i - 1, key)
        // Shift and insert
    }
}
```

## Related Concepts

- **Selection Algorithms**: QuickSelect for finding k-th smallest in O(n)
- **Partial Sorting**: HeapSelect for finding k smallest/largest
- **Topological Sort**: DAG ordering
- **External Sorting**: Disk-based sorting for massive datasets
- **Parallel Sorting**: GPU-based bitonic sort, sample sort
- **Streaming Algorithms**: Sorting data too large for memory
- **Comparison Networks**: Fixed comparison sequences (bitonic sort)

## Further Reading

- Knuth, Donald. "The Art of Computer Programming, Volume 3: Sorting and Searching"
- Sedgewick, Robert. "Algorithms in C"
- Cormen et al. "Introduction to Algorithms" (CLRS)
- Python's Timsort description: https://github.com/python/cpython/blob/main/Objects/listsort.txt

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
