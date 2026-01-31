# String Algorithms

## Overview

String algorithms are fundamental techniques for pattern matching, text processing, and string manipulation. This document covers the major algorithms for finding patterns in text, from the naive approach to sophisticated linear-time solutions. These algorithms are essential for text editors, search engines, bioinformatics, and many other applications.

## Pattern Matching Algorithms

### Naive Algorithm - O(nm)

```pseudocode
function naiveSearch(text, pattern) {
    n = length(text)
    m = length(pattern)
    positions = []

    for i from 0 to n - m {
        match = true
        for j from 0 to m - 1 {
            if text[i + j] != pattern[j] {
                match = false
                break
            }
        }
        if match {
            positions.append(i)
        }
    }

    return positions
}

// Time: O(nm) worst case, O(n) average for random strings
// Space: O(1) excluding output
```

### Knuth-Morris-Pratt (KMP) - O(n + m)

KMP never re-examines characters in the text by using information from partial matches.

```pseudocode
function computeLPSArray(pattern) {
    // LPS[i] = length of longest proper prefix which is also suffix
    //          for pattern[0..i]
    m = length(pattern)
    lps = array of size m, initialized to 0

    len = 0  // Length of previous longest prefix suffix
    i = 1

    while i < m {
        if pattern[i] == pattern[len] {
            len++
            lps[i] = len
            i++
        } else {
            if len != 0 {
                len = lps[len - 1]  // Don't increment i
            } else {
                lps[i] = 0
                i++
            }
        }
    }

    return lps
}

function kmpSearch(text, pattern) {
    n = length(text)
    m = length(pattern)

    if m == 0 {
        return []
    }

    lps = computeLPSArray(pattern)
    positions = []

    i = 0  // Index in text
    j = 0  // Index in pattern

    while i < n {
        if pattern[j] == text[i] {
            i++
            j++
        }

        if j == m {
            // Pattern found at index i - j
            positions.append(i - j)
            j = lps[j - 1]
        } else if i < n and pattern[j] != text[i] {
            if j != 0 {
                j = lps[j - 1]
            } else {
                i++
            }
        }
    }

    return positions
}

// Time: O(n + m) - preprocessing O(m), search O(n)
// Space: O(m) for LPS array
```

### Rabin-Karp - O(n + m) expected

Uses rolling hash for fast substring comparison.

```pseudocode
function rabinKarpSearch(text, pattern) {
    n = length(text)
    m = length(pattern)
    d = 256      // Number of characters in alphabet
    q = 101      // A prime number
    positions = []

    // Calculate h = d^(m-1) mod q
    h = 1
    for i from 0 to m - 2 {
        h = (h * d) mod q
    }

    // Calculate hash of pattern and first window
    patternHash = 0
    textHash = 0

    for i from 0 to m - 1 {
        patternHash = (d * patternHash + ord(pattern[i])) mod q
        textHash = (d * textHash + ord(text[i])) mod q
    }

    // Slide pattern over text
    for i from 0 to n - m {
        // Check if hashes match
        if patternHash == textHash {
            // Verify character by character (avoid spurious hits)
            match = true
            for j from 0 to m - 1 {
                if text[i + j] != pattern[j] {
                    match = false
                    break
                }
            }
            if match {
                positions.append(i)
            }
        }

        // Calculate hash for next window
        if i < n - m {
            textHash = (d * (textHash - ord(text[i]) * h) + ord(text[i + m])) mod q

            // Handle negative hash
            if textHash < 0 {
                textHash = textHash + q
            }
        }
    }

    return positions
}

// Time: O(n + m) expected, O(nm) worst case with many hash collisions
// Space: O(1)

// Rolling hash formula:
// hash(s[i+1..i+m]) = d * (hash(s[i..i+m-1]) - s[i] * d^(m-1)) + s[i+m]
```

### Boyer-Moore - O(n/m) best case

Searches from right to left, skipping large portions of text.

```pseudocode
function computeBadCharacterTable(pattern) {
    // Last occurrence of each character in pattern
    table = new Map()

    for i from 0 to length(pattern) - 1 {
        table[pattern[i]] = i
    }

    return table
}

function computeGoodSuffixTable(pattern) {
    m = length(pattern)
    suffix = computeSuffixArray(pattern)
    goodSuffix = array of size m + 1, initialized to m

    // Case 1: Matching suffix exists elsewhere in pattern
    j = 0
    for i from m - 1 down to 0 {
        if suffix[i] == i + 1 {
            // Prefix matches suffix
            while j < m - 1 - i {
                if goodSuffix[j] == m {
                    goodSuffix[j] = m - 1 - i
                }
                j++
            }
        }
    }

    // Case 2: Part of matching suffix exists at start of pattern
    for i from 0 to m - 2 {
        goodSuffix[m - 1 - suffix[i]] = m - 1 - i
    }

    return goodSuffix
}

function computeSuffixArray(pattern) {
    m = length(pattern)
    suffix = array of size m

    suffix[m - 1] = m
    g = m - 1

    for i from m - 2 down to 0 {
        if i > g and suffix[i + m - 1 - f] < i - g {
            suffix[i] = suffix[i + m - 1 - f]
        } else {
            if i < g {
                g = i
            }
            f = i
            while g >= 0 and pattern[g] == pattern[g + m - 1 - f] {
                g--
            }
            suffix[i] = f - g
        }
    }

    return suffix
}

function boyerMooreSearch(text, pattern) {
    n = length(text)
    m = length(pattern)

    if m == 0 {
        return []
    }

    badChar = computeBadCharacterTable(pattern)
    goodSuffix = computeGoodSuffixTable(pattern)
    positions = []

    i = 0  // Alignment position in text

    while i <= n - m {
        j = m - 1  // Start from end of pattern

        // Match from right to left
        while j >= 0 and pattern[j] == text[i + j] {
            j--
        }

        if j < 0 {
            // Pattern found
            positions.append(i)
            i += goodSuffix[0]
        } else {
            // Mismatch at position j
            charShift = j - badChar.get(text[i + j], -1)
            suffixShift = goodSuffix[j + 1]
            i += max(charShift, suffixShift)
        }
    }

    return positions
}

// Time: O(n/m) best case, O(nm) worst case, O(n) average
// Space: O(m + alphabet_size)
```

### Aho-Corasick - O(n + m + z)

Searches for multiple patterns simultaneously using automaton.

```pseudocode
class AhoCorasickNode {
    children = new Map()    // char -> node
    fail = null             // Failure link
    output = []             // Patterns ending at this node
}

function buildAhoCorasick(patterns) {
    root = new AhoCorasickNode()

    // Build trie
    for (patternIndex, pattern) in enumerate(patterns) {
        node = root
        for char in pattern {
            if char not in node.children {
                node.children[char] = new AhoCorasickNode()
            }
            node = node.children[char]
        }
        node.output.append(patternIndex)
    }

    // Build failure links using BFS
    queue = new Queue()
    root.fail = root

    // Initialize first level
    for child in root.children.values() {
        child.fail = root
        queue.enqueue(child)
    }

    // BFS to build failure links
    while not queue.isEmpty() {
        current = queue.dequeue()

        for (char, child) in current.children {
            queue.enqueue(child)

            // Find failure link
            fail = current.fail
            while fail != root and char not in fail.children {
                fail = fail.fail
            }

            if char in fail.children and fail.children[char] != child {
                child.fail = fail.children[char]
            } else {
                child.fail = root
            }

            // Merge output from failure chain
            child.output.extend(child.fail.output)
        }
    }

    return root
}

function ahoCorasickSearch(text, root) {
    matches = []  // (position, patternIndex)
    node = root

    for (i, char) in enumerate(text) {
        // Follow failure links until match or root
        while node != root and char not in node.children {
            node = node.fail
        }

        if char in node.children {
            node = node.children[char]
        }

        // Report matches
        for patternIndex in node.output {
            // Pattern ends at position i
            matches.append((i, patternIndex))
        }
    }

    return matches
}

// Time: O(n + m + z) where n = text length, m = total pattern length, z = matches
// Space: O(m) for automaton
```

## Comparison Table

| Algorithm    | Preprocess | Search      | Best Case  | Space     |
|--------------|------------|-------------|------------|-----------|
| Naive        | O(1)       | O(nm)       | O(n)       | O(1)      |
| KMP          | O(m)       | O(n)        | O(n)       | O(m)      |
| Rabin-Karp   | O(m)       | O(n)*       | O(n)       | O(1)      |
| Boyer-Moore  | O(m + σ)   | O(n)*       | O(n/m)     | O(m + σ)  |
| Aho-Corasick | O(m)       | O(n + z)    | O(n + z)   | O(m)      |

*Expected time, worst case O(nm)
σ = alphabet size, z = number of matches

## When to Use Each Algorithm

### Naive
- Very short patterns (m ≤ 3)
- One-time search
- Simple implementation needed

### KMP
- Single pattern, multiple texts
- Guaranteed linear time needed
- Streaming text processing

### Rabin-Karp
- Multiple patterns of same length
- Plagiarism detection
- 2D pattern matching

### Boyer-Moore
- Long patterns in natural language
- DNA/protein sequences
- When sublinear time expected

### Aho-Corasick
- Multiple patterns of different lengths
- Network intrusion detection
- Dictionary matching

## Use Cases

### Text Editor Find

```pseudocode
class TextEditor {
    function findAll(text, pattern, caseSensitive = true) {
        if not caseSensitive {
            text = lowercase(text)
            pattern = lowercase(pattern)
        }

        if length(pattern) < 4 {
            return naiveSearch(text, pattern)
        } else {
            return boyerMooreSearch(text, pattern)
        }
    }
}
```

### Spam Filter

```pseudocode
class SpamFilter {
    automaton

    function initialize(spamWords) {
        this.automaton = buildAhoCorasick(spamWords)
    }

    function isSpam(message) {
        matches = ahoCorasickSearch(message, this.automaton)
        return length(matches) > threshold
    }
}
```

### DNA Sequence Matching

```pseudocode
function findGenePatterns(genome, genes) {
    automaton = buildAhoCorasick(genes)
    matches = ahoCorasickSearch(genome, automaton)

    // Group by gene
    geneLocations = new Map()
    for (pos, geneIndex) in matches {
        gene = genes[geneIndex]
        if gene not in geneLocations {
            geneLocations[gene] = []
        }
        geneLocations[gene].append(pos)
    }

    return geneLocations
}
```

## Common Pitfalls

- **Hash collisions in Rabin-Karp**: Always verify matches
- **LPS array off-by-one**: Careful with indices
- **Boyer-Moore shift calculation**: Can be tricky to implement correctly
- **Aho-Corasick failure links**: Must follow to root completely
- **Case sensitivity**: Normalize before searching
- **Unicode handling**: Character boundaries may vary

## Related Algorithms

- **Z-Algorithm**: Linear-time pattern matching alternative to KMP
- **Suffix Array/Tree**: Preprocess text for multiple pattern searches
- **Regular Expressions**: Pattern matching with wildcards and repetition
- **Approximate Matching**: Edit distance-based matching (Levenshtein)
- **Bitap Algorithm**: Shift-or for approximate matching

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
