# Bloom Filter

## Overview

A Bloom filter is a space-efficient probabilistic data structure that tests whether an element is a member of a set. It can have false positives (saying an element is present when it isn't) but never false negatives (if it says an element is absent, it definitely is). This tradeoff enables massive space savings compared to storing actual elements.

## Properties

- **Probabilistic**: May have false positives, never false negatives
- **Space efficient**: Uses bit array instead of storing elements
- **No deletion**: Standard Bloom filter doesn't support removal
- **Fixed size**: Size determined at creation, doesn't grow
- **Multiple hash functions**: Uses k independent hash functions
- **One-way**: Cannot retrieve elements, only test membership

### False Positive Probability

```
p ≈ (1 - e^(-kn/m))^k

Where:
  k = number of hash functions
  m = size of bit array
  n = number of elements inserted
  e = Euler's number (≈2.718)

Optimal k = (m/n) * ln(2) ≈ 0.693 * (m/n)
```

## Time Complexity

| Operation | Complexity |
|-----------|------------|
| Insert    | O(k)       |
| Lookup    | O(k)       |
| Union     | O(m)       |
| Intersect | O(m)       |

Where k = number of hash functions, m = bit array size

## Space Complexity

O(m) bits where m is the bit array size. Typically 8-10 bits per element for 1% false positive rate.

```
For 1% false positive rate: m ≈ 9.6n bits
For 0.1% false positive rate: m ≈ 14.4n bits

Compare to hash set: ~64n bits (assuming 64-bit pointers)
```

## Operations

### Insert

Set k bits corresponding to k hash values:

```pseudocode
function insert(element) {
    for i from 0 to k - 1 {
        index = hash_i(element) % m
        bitArray[index] = 1
    }
}
```

### Lookup (Test Membership)

Check if all k bits are set:

```pseudocode
function contains(element) {
    for i from 0 to k - 1 {
        index = hash_i(element) % m
        if bitArray[index] == 0 {
            return false  // Definitely not present
        }
    }
    return true  // Possibly present (may be false positive)
}
```

### Union

OR two Bloom filters:

```pseudocode
function union(filter1, filter2) {
    if filter1.m != filter2.m or filter1.k != filter2.k {
        throw IncompatibleFiltersError
    }

    result = new BloomFilter(filter1.m, filter1.k)
    for i from 0 to m - 1 {
        result.bitArray[i] = filter1.bitArray[i] OR filter2.bitArray[i]
    }
    return result
}
```

### Intersection Approximation

AND two Bloom filters (approximation):

```pseudocode
function intersection(filter1, filter2) {
    result = new BloomFilter(filter1.m, filter1.k)
    for i from 0 to m - 1 {
        result.bitArray[i] = filter1.bitArray[i] AND filter2.bitArray[i]
    }
    return result
}
```

## Implementation

```pseudocode
class BloomFilter {
    bitArray[]      // Bit array of size m
    m               // Size of bit array
    k               // Number of hash functions
    n = 0           // Number of elements inserted (for estimation)

    function constructor(expectedElements, falsePositiveRate = 0.01) {
        // Calculate optimal m and k
        this.m = calculateOptimalSize(expectedElements, falsePositiveRate)
        this.k = calculateOptimalHashCount(this.m, expectedElements)
        this.bitArray = new BitArray(this.m)
    }

    function calculateOptimalSize(n, p) {
        // m = -n * ln(p) / (ln(2)^2)
        return ceil(-n * ln(p) / (ln(2) * ln(2)))
    }

    function calculateOptimalHashCount(m, n) {
        // k = (m/n) * ln(2)
        return round((m / n) * ln(2))
    }

    // Generate k hash values using double hashing technique
    function getHashValues(element) {
        hash1 = murmurHash(element, seed1)
        hash2 = murmurHash(element, seed2)

        hashes = new Array(this.k)
        for i from 0 to this.k - 1 {
            // h_i(x) = h1(x) + i * h2(x)
            hashes[i] = (hash1 + i * hash2) % this.m
            if hashes[i] < 0 {
                hashes[i] = hashes[i] + this.m
            }
        }
        return hashes
    }

    function add(element) {
        hashes = getHashValues(element)

        for index in hashes {
            this.bitArray.set(index, 1)
        }

        this.n = this.n + 1
    }

    function mightContain(element) {
        hashes = getHashValues(element)

        for index in hashes {
            if this.bitArray.get(index) == 0 {
                return false
            }
        }

        return true
    }

    function estimatedFalsePositiveRate() {
        // Actual false positive rate based on fill ratio
        bitsSet = this.bitArray.countOnes()
        fillRatio = bitsSet / this.m
        return pow(fillRatio, this.k)
    }

    function estimatedElementCount() {
        // Estimate number of elements from bit density
        bitsSet = this.bitArray.countOnes()
        if bitsSet == 0 {
            return 0
        }
        if bitsSet == this.m {
            return infinity
        }
        return -this.m * ln(1 - bitsSet / this.m) / this.k
    }

    function clear() {
        this.bitArray.clearAll()
        this.n = 0
    }

    function union(other) {
        if this.m != other.m or this.k != other.k {
            throw IncompatibleFiltersError
        }

        result = new BloomFilter()
        result.m = this.m
        result.k = this.k
        result.bitArray = this.bitArray.or(other.bitArray)
        return result
    }
}
```

## Variants

### Counting Bloom Filter

Supports deletion by using counters instead of bits:

```pseudocode
class CountingBloomFilter {
    counters[]      // Array of small integers (typically 4 bits)

    function add(element) {
        hashes = getHashValues(element)
        for index in hashes {
            if counters[index] < MAX_COUNT {
                counters[index] = counters[index] + 1
            }
        }
    }

    function remove(element) {
        if not mightContain(element) {
            return false
        }

        hashes = getHashValues(element)
        for index in hashes {
            if counters[index] > 0 {
                counters[index] = counters[index] - 1
            }
        }
        return true
    }

    function mightContain(element) {
        hashes = getHashValues(element)
        for index in hashes {
            if counters[index] == 0 {
                return false
            }
        }
        return true
    }
}
```

### Scalable Bloom Filter

Grows by adding new filters:

```pseudocode
class ScalableBloomFilter {
    filters = []        // List of bloom filters
    growthRate = 2      // Each new filter is 2x size
    errorTightening = 0.5  // Each filter has tighter error rate

    function add(element) {
        if filters.isEmpty() or lastFilter().isFull() {
            addNewFilter()
        }
        filters.last().add(element)
    }

    function mightContain(element) {
        for filter in filters {
            if filter.mightContain(element) {
                return true
            }
        }
        return false
    }

    function addNewFilter() {
        newError = initialError * pow(errorTightening, filters.length)
        newSize = initialSize * pow(growthRate, filters.length)
        filters.append(new BloomFilter(newSize, newError))
    }
}
```

### Cuckoo Filter

Better alternative with deletion support:

```pseudocode
// Stores fingerprints instead of just bits
// Supports efficient deletion
// Better space efficiency for low false positive rates
```

## Use Cases

- **Web caching**: Check if URL might be cached before disk lookup
- **Database queries**: Avoid expensive disk reads for non-existent keys
- **Spell checkers**: Quick "might be misspelled" check
- **Network routing**: Packet deduplication, loop detection
- **Distributed systems**: Check if data exists on other nodes
- **Malware detection**: Quick signature matching
- **CDN**: Content existence check across edge servers
- **Password checking**: "Have I been pwned?" style checks

## Advantages

- **Extremely space efficient**: Often 8-10 bits per element
- **Constant time**: O(k) operations regardless of element count
- **Simple**: Easy to implement and understand
- **Parallelizable**: Hash computations are independent
- **Set operations**: Union is trivial (OR the arrays)

## Disadvantages

- **False positives**: May incorrectly report membership
- **No deletion**: Standard version doesn't support removal
- **No enumeration**: Cannot list elements
- **Fixed capacity**: Must estimate size upfront
- **No count**: Cannot count occurrences
- **Accuracy degrades**: False positive rate increases with fill

## Comparison with Alternatives

| Aspect           | Bloom Filter | Hash Set   | Cuckoo Filter | Bit Set      |
|------------------|--------------|------------|---------------|--------------|
| Space per elem   | 8-10 bits    | 64+ bits   | ~12 bits      | 1 bit*       |
| False positives  | Configurable | None       | Configurable  | None         |
| False negatives  | None         | None       | Possible**    | None         |
| Deletion         | No           | Yes        | Yes           | Yes          |
| Lookup time      | O(k)         | O(1)       | O(1)          | O(1)         |
| Element types    | Any hashable | Any hashable| Any hashable | Integers     |

*Bit set only works for bounded integer ranges
**Cuckoo filter can have false negatives during heavy deletion

## Common Pitfalls

- **Underestimating size**: Filter fills up, high false positive rate
- **Overestimating size**: Wastes memory
- **Wrong k value**: Too few or too many hash functions
- **Poor hash functions**: Not independent, causes clustering
- **Ignoring false positives**: Application must handle them
- **Expecting deletion**: Standard filters don't support it
- **Not checking compatibility**: Union requires same m and k

## Design Guidelines

| False Positive Rate | Bits per Element | Hash Functions |
|---------------------|------------------|----------------|
| 10%                 | 4.8              | 3              |
| 1%                  | 9.6              | 7              |
| 0.1%                | 14.4             | 10             |
| 0.01%               | 19.2             | 13             |

## Related Structures

- **Counting Bloom Filter**: Supports deletion with counters
- **Cuckoo Filter**: Better deletion support, lower space
- **Quotient Filter**: Cache-friendly alternative
- **HyperLogLog**: Cardinality estimation
- **MinHash**: Similarity estimation
- **Skip Bloom Filter**: Supports range queries

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
