# Suffix Tree

## Overview

A Suffix Tree is a compressed trie containing all suffixes of a given string. It provides O(m) pattern matching where m is the pattern length, making it one of the most powerful data structures for string processing. Suffix trees enable linear-time solutions to many complex string problems including longest repeated substring, longest common substring, and string matching with wildcards.

The key insight is that any substring of a text is a prefix of some suffix, so searching for a pattern reduces to traversing from the root following the pattern characters.

## Properties

- **Compressed trie**: Edges labeled with substrings, not single characters
- **n leaves**: Exactly n leaves for a string of length n (with sentinel)
- **At most n-1 internal nodes**: Efficient space usage
- **O(n) construction**: Ukkonen's algorithm builds in linear time
- **O(m) pattern search**: m = pattern length
- **Implicit suffix links**: Enable linear-time construction

## Time Complexity

| Operation                      | Time      | Notes                           |
|--------------------------------|-----------|--------------------------------|
| Construction (Ukkonen)         | O(n)      | Online, left-to-right          |
| Construction (McCreight)       | O(n)      | Offline algorithm              |
| Pattern Search                 | O(m)      | m = pattern length             |
| Find All Occurrences           | O(m + k)  | k = number of occurrences      |
| Longest Repeated Substring     | O(n)      | Deepest internal node          |
| Longest Common Substring       | O(n + m)  | Two strings concatenated       |
| Longest Palindrome             | O(n)      | Using generalized suffix tree  |

## Space Complexity

O(n) nodes and edges. In practice, ~20-40 bytes per character due to pointers.

## Structure

### Node and Edge Structure

```pseudocode
class SuffixTreeNode {
    children = new Map()    // char -> SuffixTreeNode
    suffixLink = null       // For construction algorithm
    start                   // Start index of edge label
    end                     // End index of edge label (pointer for leaves)
    suffixIndex = -1        // For leaves: starting position of suffix
}

class SuffixTree {
    root
    text
    size

    // For Ukkonen's algorithm
    activeNode
    activeEdge
    activeLength
    remainingSuffixCount
    leafEnd              // Global end for all leaves
}

// Edge label is text[node.start..node.end]
// For leaves, end points to global leafEnd (trick for O(1) extension)
```

### Edge Label Representation

```pseudocode
function getEdgeLength(node) {
    return node.end - node.start + 1
}

function getEdgeLabel(tree, node) {
    return tree.text[node.start..node.end]
}

// Edges store (start, end) indices, not actual strings
// Saves space: O(1) per edge instead of O(edge_length)
```

## Ukkonen's Algorithm - O(n)

### Key Concepts

1. **Implicit suffix tree**: Tree for text[0..i] built incrementally
2. **Active point**: (activeNode, activeEdge, activeLength) tracks position
3. **Suffix links**: Shortcut for moving between suffix insertion points
4. **Rule 1**: Extend existing leaf (automatic via leafEnd pointer)
5. **Rule 2**: Create new leaf when no matching edge
6. **Rule 3**: Do nothing when suffix already exists (showstopper)

### Construction

```pseudocode
function buildSuffixTree(text) {
    tree = new SuffixTree()
    tree.text = text + "$"  // Add unique terminator
    tree.size = length(tree.text)
    tree.root = new SuffixTreeNode()
    tree.root.start = -1
    tree.root.end = -1

    // Initialize active point
    tree.activeNode = tree.root
    tree.activeEdge = -1
    tree.activeLength = 0
    tree.remainingSuffixCount = 0
    tree.leafEnd = -1

    // Build tree character by character
    for i from 0 to tree.size - 1 {
        extendSuffixTree(tree, i)
    }

    setSuffixIndices(tree.root, 0, tree)

    return tree
}

function extendSuffixTree(tree, pos) {
    // Extend leafEnd for all existing leaves (Rule 1)
    tree.leafEnd = pos
    tree.remainingSuffixCount++
    lastNewNode = null

    while tree.remainingSuffixCount > 0 {
        if tree.activeLength == 0 {
            tree.activeEdge = pos
        }

        char = tree.text[tree.activeEdge]

        // No edge starting with activeEdge char
        if char not in tree.activeNode.children {
            // Rule 2: Create new leaf
            newNode = new SuffixTreeNode()
            newNode.start = pos
            newNode.end = tree.leafEnd  // Points to global end
            newNode.suffixIndex = -1
            tree.activeNode.children[char] = newNode

            if lastNewNode != null {
                lastNewNode.suffixLink = tree.activeNode
                lastNewNode = null
            }
        } else {
            // Edge exists, walk down if needed
            next = tree.activeNode.children[char]

            if walkDown(tree, next) {
                continue  // Active point changed, re-process
            }

            // Rule 3: Character already on edge
            if tree.text[next.start + tree.activeLength] == tree.text[pos] {
                if lastNewNode != null and tree.activeNode != tree.root {
                    lastNewNode.suffixLink = tree.activeNode
                    lastNewNode = null
                }
                tree.activeLength++
                break  // Stop this phase
            }

            // Rule 2: Split edge and create new leaf
            splitNode = new SuffixTreeNode()
            splitNode.start = next.start
            splitNode.end = next.start + tree.activeLength - 1

            tree.activeNode.children[char] = splitNode
            splitNode.children[tree.text[pos]] = createLeaf(tree, pos)

            next.start = next.start + tree.activeLength
            splitNode.children[tree.text[next.start]] = next

            if lastNewNode != null {
                lastNewNode.suffixLink = splitNode
            }
            lastNewNode = splitNode
        }

        tree.remainingSuffixCount--

        if tree.activeNode == tree.root and tree.activeLength > 0 {
            tree.activeLength--
            tree.activeEdge = pos - tree.remainingSuffixCount + 1
        } else if tree.activeNode != tree.root {
            tree.activeNode = tree.activeNode.suffixLink or tree.root
        }
    }
}

function walkDown(tree, node) {
    edgeLength = getEdgeLength(node)
    if tree.activeLength >= edgeLength {
        tree.activeEdge += edgeLength
        tree.activeLength -= edgeLength
        tree.activeNode = node
        return true
    }
    return false
}

function createLeaf(tree, pos) {
    leaf = new SuffixTreeNode()
    leaf.start = pos
    leaf.end = tree.leafEnd
    return leaf
}

function setSuffixIndices(node, labelHeight, tree) {
    if node == null {
        return
    }

    isLeaf = true
    for child in node.children.values() {
        isLeaf = false
        setSuffixIndices(child, labelHeight + getEdgeLength(child), tree)
    }

    if isLeaf {
        node.suffixIndex = tree.size - labelHeight
    }
}
```

## Operations

### Pattern Search

```pseudocode
function search(tree, pattern) {
    node = tree.root
    i = 0
    m = length(pattern)

    while i < m {
        char = pattern[i]

        if char not in node.children {
            return null  // Pattern not found
        }

        child = node.children[char]
        edgeLabel = getEdgeLabel(tree, child)
        j = 0

        // Match along edge
        while j < length(edgeLabel) and i < m {
            if pattern[i] != edgeLabel[j] {
                return null  // Mismatch
            }
            i++
            j++
        }

        if i == m {
            return child  // Pattern found, return subtree root
        }

        node = child
    }

    return node
}

function findAllOccurrences(tree, pattern) {
    node = search(tree, pattern)

    if node == null {
        return []
    }

    // Collect all suffix indices in subtree
    positions = []
    collectLeaves(node, positions)
    return positions
}

function collectLeaves(node, positions) {
    if node.suffixIndex >= 0 {
        positions.append(node.suffixIndex)
        return
    }

    for child in node.children.values() {
        collectLeaves(child, positions)
    }
}
```

### Longest Repeated Substring

```pseudocode
function longestRepeatedSubstring(tree) {
    maxDepth = 0
    result = ""

    function dfs(node, depth) {
        if node.suffixIndex >= 0 {
            return  // Leaf node
        }

        // Internal node = substring appears more than once
        if depth > maxDepth {
            maxDepth = depth
            result = tree.text[0..depth-1]  // Reconstruct string
        }

        for child in node.children.values() {
            dfs(child, depth + getEdgeLength(child))
        }
    }

    dfs(tree.root, 0)
    return result
}

// Alternative: Track path during traversal
function longestRepeatedSubstringWithPath(tree) {
    maxDepth = 0
    deepestNode = null

    function dfs(node, depth) {
        if node.suffixIndex >= 0 {
            return
        }

        if depth > maxDepth {
            maxDepth = depth
            deepestNode = node
        }

        for child in node.children.values() {
            dfs(child, depth + getEdgeLength(child))
        }
    }

    dfs(tree.root, 0)

    if deepestNode == null {
        return ""
    }

    // Reconstruct from any leaf in subtree
    leaf = findAnyLeaf(deepestNode)
    return tree.text[leaf.suffixIndex..leaf.suffixIndex + maxDepth - 1]
}
```

### Longest Common Substring

```pseudocode
function longestCommonSubstring(text1, text2) {
    // Build generalized suffix tree
    combined = text1 + "#" + text2 + "$"
    tree = buildSuffixTree(combined)

    n1 = length(text1)
    maxDepth = 0
    result = ""

    function dfs(node, depth) {
        if node.suffixIndex >= 0 {
            // Leaf: determine which string
            if node.suffixIndex < n1 {
                return (true, false)  // From text1
            } else {
                return (false, true)  // From text2
            }
        }

        hasText1 = false
        hasText2 = false

        for child in node.children.values() {
            (childHas1, childHas2) = dfs(child, depth + getEdgeLength(child))
            hasText1 = hasText1 or childHas1
            hasText2 = hasText2 or childHas2
        }

        // Internal node with descendants from both strings
        if hasText1 and hasText2 and depth > maxDepth {
            maxDepth = depth
            // Reconstruct substring
            leaf = findAnyLeaf(node)
            result = combined[leaf.suffixIndex..leaf.suffixIndex + depth - 1]
        }

        return (hasText1, hasText2)
    }

    dfs(tree.root, 0)
    return result
}
```

## Use Cases

### String Matching with Wildcards

```pseudocode
function searchWithWildcard(tree, pattern) {
    // Pattern may contain '?' for single character wildcard
    results = []

    function searchHelper(node, depth, patternIndex) {
        if patternIndex == length(pattern) {
            collectLeaves(node, results)
            return
        }

        if pattern[patternIndex] == '?' {
            // Try all children
            for child in node.children.values() {
                // Match single character
                searchHelper(child, depth + 1, patternIndex + 1)
            }
        } else {
            char = pattern[patternIndex]
            if char in node.children {
                child = node.children[char]
                // Continue matching along edge
                searchHelper(child, depth + 1, patternIndex + 1)
            }
        }
    }

    searchHelper(tree.root, 0, 0)
    return results
}
```

### DNA/Protein Sequence Analysis

```pseudocode
function findRepeatedMotifs(genome, minLength) {
    tree = buildSuffixTree(genome)
    motifs = []

    function dfs(node, depth) {
        if node.suffixIndex >= 0 {
            return 1  // One occurrence
        }

        count = 0
        for child in node.children.values() {
            count += dfs(child, depth + getEdgeLength(child))
        }

        // Internal node with sufficient depth and multiple occurrences
        if depth >= minLength and count >= 2 {
            motif = reconstructString(node, depth, tree)
            motifs.append((motif, count))
        }

        return count
    }

    dfs(tree.root, 0)
    return motifs
}
```

## Advantages

- **O(m) pattern search**: Optimal for pattern matching
- **Linear construction**: Ukkonen's algorithm
- **Versatile**: Solves many string problems efficiently
- **Online construction**: Can build incrementally
- **Rich information**: Suffix links, LCP info embedded

## Disadvantages

- **High memory usage**: ~20-40 bytes per character
- **Complex implementation**: Ukkonen's algorithm is intricate
- **Cache unfriendly**: Pointer-heavy structure
- **Overkill for simple tasks**: Suffix array may suffice
- **Construction complexity**: Edge cases in Ukkonen's

## Comparison with Suffix Array

| Aspect              | Suffix Tree       | Suffix Array      |
|---------------------|-------------------|-------------------|
| Space               | O(n) but high constant | O(n) integers |
| Construction        | O(n)              | O(n) to O(n log n)|
| Pattern search      | O(m)              | O(m log n)        |
| Implementation      | Complex           | Simpler           |
| Cache efficiency    | Poor              | Good              |
| Memory (practical)  | 20-40 bytes/char  | 4-8 bytes/char    |

## Common Pitfalls

- **Missing terminator**: Must add unique character at end
- **Suffix link management**: Complex during construction
- **Edge splitting errors**: Must maintain correct indices
- **Memory leaks**: Many nodes allocated
- **Off-by-one in edge labels**: Inclusive vs exclusive bounds
- **Leaf end pointer**: Must use global variable trick

## Related Structures

- **Suffix Array**: Space-efficient alternative
- **Suffix Automaton**: Minimal DFA accepting all suffixes
- **Generalized Suffix Tree**: Multiple strings in one tree
- **Compressed Suffix Tree**: Reduced space representation
- **FM-Index**: Compressed full-text index

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
