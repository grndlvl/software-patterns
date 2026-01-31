# Suffix Array

## Overview

A Suffix Array is a sorted array of all suffixes of a string. It provides a space-efficient alternative to Suffix Trees while supporting similar functionality for string processing tasks. Given a string S of length n, the suffix array SA contains integers 0 to n-1, where SA[i] is the starting position of the i-th smallest suffix in lexicographical order.

Suffix arrays are fundamental for string matching, text indexing, and bioinformatics applications. Combined with the LCP (Longest Common Prefix) array, they can solve most problems that suffix trees can solve.

## Properties

- **Sorted suffixes**: All suffixes arranged in lexicographical order
- **Space efficient**: O(n) integers vs O(n) nodes in suffix tree
- **Cache friendly**: Array-based structure with good locality
- **Binary searchable**: O(m log n) pattern matching
- **Foundation for BWT**: Used in Burrows-Wheeler Transform

## Time Complexity

| Operation                    | Time            | Notes                           |
|------------------------------|-----------------|--------------------------------|
| Construction (Naive)         | O(n² log n)     | Sort all suffixes              |
| Construction (Prefix Doubling)| O(n log n)     | Manber-Myers algorithm         |
| Construction (DC3/SA-IS)     | O(n)            | Linear time algorithms         |
| LCP Array (Kasai)            | O(n)            | After SA is built              |
| Pattern Search               | O(m log n)      | Binary search                  |
| Pattern Search (with LCP)    | O(m + log n)    | Enhanced with LCP              |
| All Occurrences              | O(m log n + k)  | k = number of occurrences      |

## Space Complexity

O(n) for the suffix array. Additional O(n) for LCP array if needed.

## Construction Algorithms

### Naive Construction - O(n² log n)

```pseudocode
function buildSuffixArrayNaive(text) {
    n = length(text)
    suffixes = []

    // Create (suffix, starting_position) pairs
    for i from 0 to n - 1 {
        suffixes.append((text[i..n-1], i))
    }

    // Sort by suffix string
    sort(suffixes by first element)

    // Extract starting positions
    SA = [pos for (suffix, pos) in suffixes]
    return SA
}

// Time: O(n² log n) - sorting n suffixes, each comparison O(n)
// Space: O(n²) - storing all suffixes explicitly
```

### Prefix Doubling (Manber-Myers) - O(n log n)

```pseudocode
function buildSuffixArrayPrefixDoubling(text) {
    n = length(text)
    SA = [0, 1, ..., n-1]  // Initial positions
    rank = [ord(text[i]) for i in 0..n-1]
    tmp = array of size n
    k = 1

    while k < n {
        // Compare by (rank[i], rank[i+k]) pairs
        function compare(i, j) {
            if rank[i] != rank[j] {
                return rank[i] - rank[j]
            }
            ri = rank[i + k] if i + k < n else -1
            rj = rank[j + k] if j + k < n else -1
            return ri - rj
        }

        // Sort SA using comparison function
        sort(SA using compare)

        // Recompute ranks
        tmp[SA[0]] = 0
        for i from 1 to n - 1 {
            tmp[SA[i]] = tmp[SA[i-1]]
            if compare(SA[i-1], SA[i]) < 0 {
                tmp[SA[i]] += 1
            }
        }

        rank = copy(tmp)
        k = k * 2
    }

    return SA
}

// Key insight: Compare 2k characters using two rank lookups
// Each iteration doubles the effective comparison length
```

### DC3 Algorithm - O(n)

```pseudocode
function buildSuffixArrayDC3(text) {
    n = length(text)

    // Step 1: Partition suffixes by position mod 3
    // B0: positions 0, 3, 6, ...
    // B1: positions 1, 4, 7, ...
    // B2: positions 2, 5, 8, ...

    // Step 2: Sort B1 ∪ B2 recursively
    // Create triplets for positions i where i mod 3 != 0
    R = [i for i in range(n) if i % 3 != 0]
    SA12 = radixSortTriplets(text, R)

    // If not unique, recurse on reduced string
    if not allRanksUnique(SA12) {
        reducedString = createReducedString(SA12)
        SA12 = buildSuffixArrayDC3(reducedString)
        SA12 = mapBackToOriginal(SA12)
    }

    // Step 3: Sort B0 using SA12
    SA0 = sortB0UsingSA12(text, SA12)

    // Step 4: Merge SA0 and SA12
    return merge(SA0, SA12, text)
}

// Recurrence: T(n) = T(2n/3) + O(n) = O(n)
```

### SA-IS Algorithm - O(n)

```pseudocode
function buildSuffixArraySAIS(text) {
    n = length(text)

    // Classify each position as S-type or L-type
    // S-type: text[i] < text[i+1] or (equal and i+1 is S-type)
    // L-type: otherwise
    types = classifyTypes(text)

    // Find LMS (Left-Most S-type) positions
    // LMS: S-type position preceded by L-type
    lms = findLMSPositions(types)

    // Induced sort LMS substrings
    SA = inducedSort(text, types, lms)

    // If LMS substrings not unique, recurse
    if not allUnique(lmsSA) {
        reducedText = createReducedString(lmsSA)
        reducedSA = buildSuffixArraySAIS(reducedText)
        lmsSA = mapBack(reducedSA, lms)
    }

    // Final induced sort
    return inducedSort(text, types, lmsSA)
}

function inducedSort(text, types, lmsSA) {
    SA = array of -1, size n
    buckets = computeBuckets(text)

    // Place LMS suffixes at end of buckets
    for pos in reversed(lmsSA) {
        char = text[pos]
        SA[buckets[char].end--] = pos
    }

    // Induce L-type (left to right)
    for i from 0 to n - 1 {
        if SA[i] > 0 and types[SA[i] - 1] == L_TYPE {
            char = text[SA[i] - 1]
            SA[buckets[char].start++] = SA[i] - 1
        }
    }

    // Induce S-type (right to left)
    for i from n - 1 down to 0 {
        if SA[i] > 0 and types[SA[i] - 1] == S_TYPE {
            char = text[SA[i] - 1]
            SA[buckets[char].end--] = SA[i] - 1
        }
    }

    return SA
}
```

## LCP Array (Longest Common Prefix)

The LCP array stores the length of the longest common prefix between consecutive suffixes in the sorted order.

```
LCP[i] = length of LCP between suffixes at SA[i] and SA[i-1]
LCP[0] = 0 (by convention)
```

### Kasai's Algorithm - O(n)

```pseudocode
function computeLCPArray(text, SA) {
    n = length(text)
    LCP = array of size n
    rank = array of size n

    // Compute inverse suffix array (rank)
    for i from 0 to n - 1 {
        rank[SA[i]] = i
    }

    h = 0  // Current LCP length

    // Process suffixes in text order
    for i from 0 to n - 1 {
        if rank[i] > 0 {
            j = SA[rank[i] - 1]  // Previous suffix in SA order

            // Extend LCP
            while i + h < n and j + h < n and text[i + h] == text[j + h] {
                h = h + 1
            }

            LCP[rank[i]] = h

            // Key: LCP can only decrease by 1 when moving in text order
            if h > 0 {
                h = h - 1
            }
        }
    }

    return LCP
}

// Key insight: Process in text order, not SA order
// h decreases by at most 1 per iteration, total increases bounded by n
```

## Operations

### Pattern Matching

```pseudocode
function searchPattern(text, SA, pattern) {
    n = length(SA)
    m = length(pattern)

    // Binary search for lower bound
    left = 0
    right = n

    while left < right {
        mid = (left + right) / 2
        suffix = text[SA[mid]..]

        if suffix < pattern {
            left = mid + 1
        } else {
            right = mid
        }
    }
    lower = left

    // Binary search for upper bound
    left = 0
    right = n

    while left < right {
        mid = (left + right) / 2
        suffix = text[SA[mid]..SA[mid] + m]

        if suffix <= pattern {
            left = mid + 1
        } else {
            right = mid
        }
    }
    upper = left

    // Return all positions
    if lower < upper {
        return SA[lower..upper-1]
    }
    return []
}
```

### Longest Repeated Substring

```pseudocode
function longestRepeatedSubstring(text, SA, LCP) {
    maxLCP = 0
    position = -1

    for i from 1 to length(LCP) - 1 {
        if LCP[i] > maxLCP {
            maxLCP = LCP[i]
            position = SA[i]
        }
    }

    if maxLCP == 0 {
        return ""
    }

    return text[position..position + maxLCP - 1]
}
```

### Longest Common Substring of Two Strings

```pseudocode
function longestCommonSubstring(text1, text2) {
    // Concatenate with separator
    combined = text1 + "#" + text2 + "$"

    SA = buildSuffixArray(combined)
    LCP = computeLCPArray(combined, SA)

    n1 = length(text1)
    maxLCP = 0
    position = -1

    // Find max LCP where adjacent suffixes are from different strings
    for i from 1 to length(SA) - 1 {
        fromDifferent = (SA[i] < n1) != (SA[i-1] < n1)

        if fromDifferent and LCP[i] > maxLCP {
            maxLCP = LCP[i]
            position = SA[i]
        }
    }

    if maxLCP == 0 {
        return ""
    }

    return combined[position..position + maxLCP - 1]
}
```

### Count Distinct Substrings

```pseudocode
function countDistinctSubstrings(text) {
    n = length(text)
    SA = buildSuffixArray(text)
    LCP = computeLCPArray(text, SA)

    // Total possible substrings
    total = n * (n + 1) / 2

    // Subtract duplicates (sum of LCP values)
    duplicates = sum(LCP)

    return total - duplicates
}
```

## Use Cases

### Text Indexing

```pseudocode
class TextIndex {
    text
    SA
    LCP

    function build(text) {
        this.text = text + "$"  // Add sentinel
        this.SA = buildSuffixArray(this.text)
        this.LCP = computeLCPArray(this.text, this.SA)
    }

    function search(pattern) {
        return searchPattern(this.text, this.SA, pattern)
    }

    function countOccurrences(pattern) {
        positions = search(pattern)
        return length(positions)
    }
}
```

### Burrows-Wheeler Transform

```pseudocode
function computeBWT(text) {
    SA = buildSuffixArray(text)
    BWT = ""

    for i in SA {
        if i == 0 {
            BWT += text[length(text) - 1]
        } else {
            BWT += text[i - 1]
        }
    }

    return BWT
}

// BWT is used in bzip2 compression
```

## Advantages

- **Space efficient**: O(n) integers vs suffix tree's O(n) nodes with pointers
- **Cache friendly**: Better locality than pointer-based structures
- **Simpler implementation**: Easier than suffix trees
- **Linear construction**: O(n) algorithms available
- **Versatile**: With LCP, solves most suffix tree problems

## Disadvantages

- **Slower pattern search**: O(m log n) vs O(m) for suffix trees
- **Requires LCP for some operations**: Additional preprocessing
- **Not optimal for all queries**: Some queries faster with suffix tree
- **Construction complexity**: Linear algorithms are complex

## Comparison with Suffix Tree

| Aspect              | Suffix Array      | Suffix Tree       |
|---------------------|-------------------|-------------------|
| Space               | O(n) integers     | O(n) nodes+edges  |
| Memory (practical)  | ~4-8 bytes/char   | ~20-40 bytes/char |
| Construction        | O(n) or O(n log n)| O(n)              |
| Pattern search      | O(m log n)        | O(m)              |
| With LCP array      | Most operations   | All operations    |
| Cache locality      | Excellent         | Poor              |
| Implementation      | Simpler           | Complex           |

## Common Pitfalls

- **Missing sentinel**: Add unique character at end ($ < all others)
- **Off-by-one in binary search**: Careful with bounds
- **Processing LCP in wrong order**: Kasai's requires text order
- **Comparing full suffixes**: Only compare needed prefix length
- **Not handling empty pattern**: Check edge cases
- **Integer overflow**: Use appropriate integer size for large texts

## Related Structures

- **Suffix Tree**: Trie of all suffixes, O(m) pattern search
- **LCP Array**: Longest common prefix between adjacent suffixes
- **FM-Index**: Compressed full-text index based on BWT
- **Suffix Automaton**: Minimal DFA accepting all suffixes
- **Enhanced Suffix Array**: SA + LCP + additional tables

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
