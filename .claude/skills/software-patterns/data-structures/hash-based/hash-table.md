# Hash Table (Hash Map)

## Overview

A hash table is a data structure that implements an associative array, mapping keys to values using a hash function. The hash function converts keys into array indices, enabling average O(1) access, insertion, and deletion. Hash tables are the foundation of dictionaries, sets, and caches in most programming languages.

## Properties

- **Key-value mapping**: Associates keys with values
- **Hash function**: Converts keys to array indices
- **Collision handling**: Multiple keys may hash to same index
- **Dynamic resizing**: Grows when load factor exceeds threshold
- **Unordered**: No guaranteed iteration order (unless using ordered variant)

### Load Factor

```
Load Factor = n / m

Where:
  n = number of entries
  m = number of buckets

Typical threshold: 0.75 (resize when exceeded)
```

## Time Complexity

| Operation | Average | Worst  |
|-----------|---------|--------|
| Search    | O(1)    | O(n)   |
| Insert    | O(1)    | O(n)   |
| Delete    | O(1)    | O(n)   |

*Worst case occurs with poor hash function or many collisions*

## Space Complexity

O(n) where n is the number of key-value pairs. Additional overhead for bucket array and collision structures.

## Hash Functions

### Properties of Good Hash Functions

1. **Deterministic**: Same key always produces same hash
2. **Uniform distribution**: Spreads keys evenly across buckets
3. **Fast computation**: O(1) time
4. **Avalanche effect**: Small input change causes large hash change

### Common Hash Functions

```pseudocode
// For integers
function hashInteger(key, tableSize) {
    return key % tableSize
}

// For strings (polynomial rolling hash)
function hashString(key, tableSize) {
    hash = 0
    prime = 31

    for char in key {
        hash = (hash * prime + charCode(char)) % tableSize
    }

    return hash
}

// Universal hashing
function universalHash(key, tableSize, a, b, p) {
    // p is prime larger than tableSize
    // a, b are random in [0, p-1], a != 0
    return ((a * key + b) % p) % tableSize
}

// Multiplicative hashing
function multiplicativeHash(key, tableSize) {
    A = 0.6180339887  // (sqrt(5) - 1) / 2
    return floor(tableSize * ((key * A) % 1))
}
```

## Collision Resolution Strategies

### 1. Separate Chaining

Each bucket contains a linked list of entries:

```pseudocode
class HashTableChaining {
    buckets[]       // Array of linked lists
    size = 0
    capacity

    function constructor(capacity = 16) {
        this.capacity = capacity
        this.buckets = new Array(capacity)
        for i from 0 to capacity - 1 {
            this.buckets[i] = new LinkedList()
        }
    }

    function hash(key) {
        return hashFunction(key) % this.capacity
    }

    function put(key, value) {
        index = hash(key)
        bucket = this.buckets[index]

        // Check if key exists
        for entry in bucket {
            if entry.key == key {
                entry.value = value
                return
            }
        }

        // Add new entry
        bucket.append(new Entry(key, value))
        this.size = this.size + 1

        // Resize if needed
        if this.size > this.capacity * LOAD_FACTOR_THRESHOLD {
            resize()
        }
    }

    function get(key) {
        index = hash(key)
        bucket = this.buckets[index]

        for entry in bucket {
            if entry.key == key {
                return entry.value
            }
        }

        return null
    }

    function remove(key) {
        index = hash(key)
        bucket = this.buckets[index]

        for entry in bucket {
            if entry.key == key {
                bucket.remove(entry)
                this.size = this.size - 1
                return entry.value
            }
        }

        return null
    }
}
```

### 2. Open Addressing (Linear Probing)

Find next available slot when collision occurs:

```pseudocode
class HashTableLinearProbing {
    keys[]
    values[]
    size = 0
    capacity
    DELETED = new Object()  // Tombstone marker

    function constructor(capacity = 16) {
        this.capacity = capacity
        this.keys = new Array(capacity).fill(null)
        this.values = new Array(capacity).fill(null)
    }

    function hash(key) {
        return hashFunction(key) % this.capacity
    }

    function put(key, value) {
        if this.size >= this.capacity * LOAD_FACTOR_THRESHOLD {
            resize()
        }

        index = hash(key)

        while this.keys[index] != null and this.keys[index] != DELETED {
            if this.keys[index] == key {
                this.values[index] = value
                return
            }
            index = (index + 1) % this.capacity
        }

        this.keys[index] = key
        this.values[index] = value
        this.size = this.size + 1
    }

    function get(key) {
        index = hash(key)
        startIndex = index

        while this.keys[index] != null {
            if this.keys[index] != DELETED and this.keys[index] == key {
                return this.values[index]
            }
            index = (index + 1) % this.capacity
            if index == startIndex {
                break
            }
        }

        return null
    }

    function remove(key) {
        index = hash(key)
        startIndex = index

        while this.keys[index] != null {
            if this.keys[index] != DELETED and this.keys[index] == key {
                value = this.values[index]
                this.keys[index] = DELETED
                this.values[index] = null
                this.size = this.size - 1
                return value
            }
            index = (index + 1) % this.capacity
            if index == startIndex {
                break
            }
        }

        return null
    }
}
```

### 3. Quadratic Probing

```pseudocode
function probe(index, attempt) {
    // Quadratic: index + c1*i + c2*i^2
    return (index + attempt + attempt * attempt) % this.capacity
}
```

### 4. Double Hashing

```pseudocode
function probe(key, attempt) {
    hash1 = primaryHash(key)
    hash2 = secondaryHash(key)
    // hash2 should never be 0; often use: hash2 = prime - (key % prime)
    return (hash1 + attempt * hash2) % this.capacity
}
```

## Implementation

### Complete Hash Table with Chaining

```pseudocode
class HashMap {
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

    function hash(key) {
        // Simple hash for demonstration
        hashCode = 0
        keyStr = toString(key)
        for char in keyStr {
            hashCode = (hashCode * 31 + charCode(char)) & 0x7FFFFFFF
        }
        return hashCode % this.capacity
    }

    function put(key, value) {
        index = hash(key)

        // Update if exists
        for entry in this.buckets[index] {
            if entry.key == key {
                oldValue = entry.value
                entry.value = value
                return oldValue
            }
        }

        // Insert new
        this.buckets[index].append({ key: key, value: value })
        this.size = this.size + 1

        // Resize check
        if this.size > this.capacity * this.LOAD_FACTOR {
            resize(this.capacity * 2)
        }

        return null
    }

    function get(key) {
        index = hash(key)

        for entry in this.buckets[index] {
            if entry.key == key {
                return entry.value
            }
        }

        return null
    }

    function remove(key) {
        index = hash(key)

        for i from 0 to this.buckets[index].length - 1 {
            if this.buckets[index][i].key == key {
                entry = this.buckets[index][i]
                this.buckets[index].removeAt(i)
                this.size = this.size - 1
                return entry.value
            }
        }

        return null
    }

    function containsKey(key) {
        return get(key) != null
    }

    function containsValue(value) {
        for bucket in this.buckets {
            for entry in bucket {
                if entry.value == value {
                    return true
                }
            }
        }
        return false
    }

    function keys() {
        result = []
        for bucket in this.buckets {
            for entry in bucket {
                result.append(entry.key)
            }
        }
        return result
    }

    function values() {
        result = []
        for bucket in this.buckets {
            for entry in bucket {
                result.append(entry.value)
            }
        }
        return result
    }

    function entries() {
        result = []
        for bucket in this.buckets {
            for entry in bucket {
                result.append(entry)
            }
        }
        return result
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
            for entry in bucket {
                put(entry.key, entry.value)
            }
        }
    }

    function size() {
        return this.size
    }

    function isEmpty() {
        return this.size == 0
    }

    function clear() {
        for i from 0 to this.capacity - 1 {
            this.buckets[i] = []
        }
        this.size = 0
    }
}
```

## Collision Strategies Comparison

| Strategy           | Pros                           | Cons                          |
|--------------------|--------------------------------|-------------------------------|
| Separate Chaining  | Simple, no clustering          | Extra memory for links        |
| Linear Probing     | Cache-friendly, simple         | Primary clustering            |
| Quadratic Probing  | Reduces clustering             | Secondary clustering          |
| Double Hashing     | Best distribution              | More computation              |

## Use Cases

- **Dictionaries/Maps**: Key-value storage in most languages
- **Caches**: O(1) lookup for cached data
- **Symbol tables**: Compiler variable/function lookup
- **Database indexing**: Quick record retrieval
- **Counting/Frequency**: Word counts, histogram
- **Deduplication**: Finding unique elements
- **Two-sum type problems**: O(n) complement lookup

## Advantages

- **O(1) average operations**: Fast insert, lookup, delete
- **Flexible keys**: Any hashable type can be key
- **Dynamic sizing**: Grows as needed
- **Simple interface**: Get, put, remove

## Disadvantages

- **Worst case O(n)**: With many collisions
- **No ordering**: Keys not sorted (use TreeMap for order)
- **Hash function quality**: Performance depends on good hashing
- **Memory overhead**: Empty buckets, load factor < 1
- **Not cache-optimal**: Chaining has scattered memory access

## Comparison with Alternatives

| Aspect          | Hash Table  | BST         | Sorted Array | Trie        |
|-----------------|-------------|-------------|--------------|-------------|
| Search          | O(1)        | O(log n)    | O(log n)     | O(m)        |
| Insert          | O(1)        | O(log n)    | O(n)         | O(m)        |
| Delete          | O(1)        | O(log n)    | O(n)         | O(m)        |
| Ordered         | No          | Yes         | Yes          | Prefix      |
| Range queries   | O(n)        | O(log n+k)  | O(log n+k)   | O(m+k)      |
| Memory          | Higher      | Lower       | Compact      | Higher      |

Where m = key length, k = result size

## Common Pitfalls

- **Mutable keys**: Changing key after insertion breaks lookup
- **Poor hash function**: Causes clustering and O(n) operations
- **Not handling null**: Null keys/values need special handling
- **Forgetting resize**: Fixed-size table becomes slow
- **Load factor too high**: Degrades to O(n)
- **Load factor too low**: Wastes memory
- **Hash collision attacks**: Malicious inputs can cause O(n) operations

## Related Structures

- **Hash Set**: Hash table storing only keys (no values)
- **Linked Hash Map**: Maintains insertion order
- **Concurrent Hash Map**: Thread-safe variant
- **Bloom Filter**: Probabilistic membership testing
- **Cuckoo Hashing**: O(1) worst-case lookup
- **Robin Hood Hashing**: Variance reduction in probe length

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
